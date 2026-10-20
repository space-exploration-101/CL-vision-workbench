# 端到端工作流精简说明

本流程只验证“数据预处理 → 单卡训练 → 量化”的软件链路，不用于精度、FPGA 或硬件验收。以下命令均在 `ssh H200` 后执行；凡调用容器内程序，均给出完整的 `docker run` 命令。

## 1. 数据集

默认使用已经发布并校验的 21 类合成数据集，不使用现有 shared datasets 或其他用户的数据：

```bash
DATASET=/data1/ywang/workflow-test/synthetic-pose21-r-only-v1/prepared
test -f "$DATASET/VERIFIED"
```

源图语义为 R-only；预处理结果为单通道 `gray1 [N,1,1280,1280]`。

## 2. 数据预处理

现有 `prepared` 数据无需重建。以下命令仅用于在一个全新目录中重跑预处理，不得覆盖现有数据。

先在 H200 宿主机设置路径并创建新目录：

```bash
RUN_ID="prep_$(date +%Y%m%d_%H%M%S)"
GRAYPREP_REPO=/data3/ywang/yolo-gray1-data-pipeline
RAW=/data1/ywang/workflow-test/synthetic-pose21-r-only-v1/raw
PREP_RUN="/data1/ywang/workflow-test/runs/$RUN_ID"

test ! -e "$PREP_RUN"
mkdir -p "$PREP_RUN"
```

确定性选择样本：

```bash
docker run --rm \
  --network none \
  --read-only \
  --user "$(id -u):$(id -g)" \
  --tmpfs /tmp:rw,noexec,nosuid,size=128m \
  -e PYTHONPATH=/app/src \
  -v "$GRAYPREP_REPO/src:/app/src:ro" \
  -v "$GRAYPREP_REPO/configs:/app/configs:ro" \
  -v "$GRAYPREP_REPO/schemas:/app/schemas:ro" \
  -v "$RAW:/source:ro" \
  -v "$PREP_RUN:/output:rw" \
  ywang/yolo-gray1-data-pipeline:0.1.1 \
  dataset select-smoke \
  --source /source \
  --output /output/selection.json \
  --seed 20260912 \
  --per-class 1
```

生成 `net1280` 数据以及测试用 `cam2000` BIN：

```bash
docker run --rm \
  --network none \
  --read-only \
  --user "$(id -u):$(id -g)" \
  --tmpfs /tmp:rw,noexec,nosuid,size=128m \
  -e PYTHONPATH=/app/src \
  -v "$GRAYPREP_REPO/src:/app/src:ro" \
  -v "$GRAYPREP_REPO/configs:/app/configs:ro" \
  -v "$GRAYPREP_REPO/schemas:/app/schemas:ro" \
  -v "$RAW:/source:ro" \
  -v "$PREP_RUN:/output:rw" \
  ywang/yolo-gray1-data-pipeline:0.1.1 \
  dataset build-smoke \
  --source /source \
  --manifest /output/selection.json \
  --output /output/prepared \
  --net-config /app/configs/net1280.yaml \
  --cam-config /app/configs/cam2000.yaml \
  --schema /app/schemas/preprocess-profile.schema.json \
  --workers 32 \
  --opencv-threads 1
```

严格校验输出：

```bash
docker run --rm \
  --network none \
  --read-only \
  --user "$(id -u):$(id -g)" \
  --tmpfs /tmp:rw,noexec,nosuid,size=128m \
  -e PYTHONPATH=/app/src \
  -v "$GRAYPREP_REPO/src:/app/src:ro" \
  -v "$PREP_RUN:/output:rw" \
  -v "$RAW:/source:ro" \
  ywang/yolo-gray1-data-pipeline:0.1.1 \
  dataset verify-smoke \
  --output /output/prepared \
  --source /source
```

如使用这次的新输出：

```bash
DATASET="$PREP_RUN/prepared"
test -f "$DATASET/VERIFIED"
```

## 3. 单卡训练

训练使用标准化单通道训练镜像：

```text
ywang/yolo-gray1-train:ultralytics-8.3.98-v1
```

该正式镜像已经包含单通道数组连续化处理和 Validator 单通道 warmup 支持。先在宿主机持续检查 GPU、显存和进程 owner，不能根据单次 `0%` 判断空闲：

```bash
nvidia-smi --query-gpu=index,uuid,name,memory.used,memory.free,utilization.gpu --format=csv
nvidia-smi dmon -s pucm -c 5
nvidia-smi --query-compute-apps=gpu_uuid,pid,process_name,used_memory --format=csv
```

选择一张确认空闲的 GPU，准备独立运行目录：

推荐直接使用已经保存的训练脚本，避免在交互式终端粘贴很长的多行命令：

```bash
/data3/ywang/yolo-gray1-train/run-gray1-training-smoke.sh \
  '<approved-idle-gpu-uuid>' \
  "$DATASET"
```

脚本会自动执行 GPU 连续采样和门禁、创建新的 `RUN_ID`/`RUN_ROOT`、生成容器数据集配置、运行 1 epoch，并检查 `best.pt`。下面保留其完整等价命令，便于审计和手工排错。

```bash
GPU_UUID='<approved-idle-gpu-uuid>'
RUN_ID="e2e_smoke_$(date +%Y%m%d_%H%M%S)"
RUN_ROOT="/data1/ywang/workflow-test/runs/$RUN_ID"

test ! -e "$RUN_ROOT"
mkdir -p "$RUN_ROOT/experiments"
cp "$DATASET/dataset.yaml" "$RUN_ROOT/dataset.container.yaml"
sed -i 's|^path:.*|path: /dataset|' "$RUN_ROOT/dataset.container.yaml"
```

运行 1 epoch，并用 55 分钟硬超时限制任务：

```bash
timeout --signal=TERM --kill-after=30s 55m \
docker run --rm \
  --name "ywang-$RUN_ID" \
  --stop-timeout 30 \
  --gpus "device=$GPU_UUID" \
  --user "$(id -u):$(id -g)" \
  --network none \
  --read-only \
  --shm-size 64g \
  --tmpfs /tmp:rw,nosuid,size=2g \
  -e HOME=/tmp \
  -e YOLO_CONFIG_DIR=/tmp/ultralytics \
  -e MPLCONFIGDIR=/tmp/matplotlib \
  -e PYTHONPATH=/workspace/src/Yolo_fine/ultralytics \
  -v "$DATASET:/dataset:ro" \
  -v "$RUN_ROOT/dataset.container.yaml:/config/dataset.yaml:ro" \
  -v "$RUN_ROOT/experiments:/experiments:rw" \
  --entrypoint python \
  ywang/yolo-gray1-train:ultralytics-8.3.98-v1 \
  /workspace/src/Yolo_fine/train_yolov8x_p6_pose_gray1.py \
  --data /config/dataset.yaml \
  --epochs 1 \
  --imgsz 1280 \
  --batch 4 \
  --device 0 \
  --workers 4 \
  --project /experiments \
  --name "$RUN_ID" \
  --save-period 1
```

确认模型产物：

```bash
MODEL="$RUN_ROOT/experiments/$RUN_ID/weights/best.pt"
test -s "$MODEL"
sha256sum "$MODEL"
```

## 4. 量化

在宿主机将本次模型与同一数据集组装为任务 ZIP：

```bash
QPKG="$RUN_ROOT/quant-input"
test ! -e "$QPKG"
mkdir -p "$QPKG/input/cali" "$QPKG/input/test/images" "$QPKG/input/test/labels"

cp "$MODEL" "$QPKG/input/model.pt"
cp "$DATASET"/images/calibration/*.png "$QPKG/input/cali/"
cp "$DATASET"/images/test/*.png "$QPKG/input/test/images/"
cp "$DATASET"/labels/test/*.txt "$QPKG/input/test/labels/"

python3 -c 'import json,sys; p={"nc":21,"imgsz":1280,"min_test_images":1,"min_cali_images":1,"preprocess_mode":"passthrough","input_channels":1,"input_semantics":"gray1","use_scratch":True}; open(sys.argv[1],"w",encoding="utf-8").write(json.dumps(p,indent=2)+"\n")' "$QPKG/job_config.json"

(cd "$QPKG" && zip -qr "$RUN_ROOT/quant-input.zip.partial" input job_config.json)
unzip -t "$RUN_ROOT/quant-input.zip.partial"
mv "$RUN_ROOT/quant-input.zip.partial" "$RUN_ROOT/quant-input.zip"
```

量化 API 本身运行在已有的项目容器中，下面的 `curl` 是宿主机命令，不需要再套 `docker run`。先检查服务和 GPU readiness：

```bash
curl -fsS http://127.0.0.1:8000/health
curl -fsS http://127.0.0.1:8000/ready
```

创建任务并解析返回的 `task_id`：

```bash
CREATE_JSON=$(curl -fsS -X POST \
  -F "display_name=$RUN_ID" \
  -F "onnx_name=$RUN_ID" \
  -F "preprocess_mode=passthrough" \
  -F "archive=@$RUN_ROOT/quant-input.zip" \
  http://127.0.0.1:8000/api/tasks)

printf '%s\n' "$CREATE_JSON"
TASK_ID=$(printf '%s' "$CREATE_JSON" | python3 -c 'import json,sys; print(json.load(sys.stdin)["task_id"])')
```

启动并查询任务：

```bash
curl -fsS -X POST "http://127.0.0.1:8000/api/tasks/$TASK_ID/start"
curl -fsS "http://127.0.0.1:8000/api/tasks/$TASK_ID" | python3 -m json.tool
```

当前 `generate_bin` 可能进入已知阻塞；必须保留真实失败结果。后续阶段应记为依赖跳过，整体最多为 `software_validated_partial`，不能声明 FPGA 或硬件验证完成。量化接口当前没有整条任务的一小时硬超时，因此不得无人值守运行。

更详细的解释和历史证据见 [`RUNBOOK.md`](RUNBOOK.md) 与 [`EXECUTION_RECORD.md`](EXECUTION_RECORD.md)。

## 附录：训练配置文件与参数来源

训练参数按下面的顺序逐级覆盖，排在后面的优先级更高：

```text
Ultralytics 默认配置
→ 模型结构和数据集配置
→ 项目训练脚本的默认值及固定值
→ docker run 命令行参数
→ 本次运行最终记录 args.yaml
```

### A. 启动命令

本次训练显式传入的参数位于本文“单卡训练”一节，例如：

```text
--epochs 1
--imgsz 1280
--batch 4
--device 0
--workers 4
--save-period 1
```

这些命令行参数会覆盖训练脚本中的同名默认值。

### B. 项目训练脚本

H200 宿主机：

```text
/data3/ywang/yolo-gray1-train/src/Yolo_fine/train_yolov8x_p6_pose_gray1.py
```

训练容器内：

```text
/workspace/src/Yolo_fine/train_yolov8x_p6_pose_gray1.py
```

该脚本定义命令行默认值，并固定 `amp`、`patience`、缓存和数据增强等项目级参数。查看宿主机文件：

```bash
sed -n '1,260p' \
  /data3/ywang/yolo-gray1-train/src/Yolo_fine/train_yolov8x_p6_pose_gray1.py
```

查看脚本支持的参数；该命令不使用 GPU：

```bash
docker run --rm \
  --network none \
  --read-only \
  --entrypoint python \
  ywang/yolo-gray1-train:ultralytics-8.3.98-v1 \
  /workspace/src/Yolo_fine/train_yolov8x_p6_pose_gray1.py \
  --help
```

### C. 模型结构配置

训练容器内路径：

```text
/workspace/src/Yolo_fine/configs/yolov8x-pose-p6-gray1.yaml
```

该文件定义单通道输入、21个类别、关键点形状以及 backbone/head 网络结构。查看命令：

```bash
docker run --rm \
  --network none \
  --read-only \
  --entrypoint sed \
  ywang/yolo-gray1-train:ultralytics-8.3.98-v1 \
  -n '1,260p' \
  /workspace/src/Yolo_fine/configs/yolov8x-pose-p6-gray1.yaml
```

必须保持的当前接口包括：

```yaml
ch: 1
nc: 21
kpt_shape: [2, 3]
```

### D. 数据集配置

原始预处理数据集配置：

```text
$DATASET/dataset.yaml
```

为本次训练生成的宿主机配置：

```text
$RUN_ROOT/dataset.container.yaml
```

它在训练容器内映射为：

```text
/config/dataset.yaml
```

查看本次配置：

```bash
cat "$RUN_ROOT/dataset.container.yaml"
```

其中 `path` 必须指向容器内只读数据挂载点：

```yaml
path: /dataset
```

### E. Ultralytics 默认配置

训练容器内路径：

```text
/workspace/src/Yolo_fine/ultralytics/ultralytics/cfg/default.yaml
```

该文件提供未被项目脚本或命令行覆盖的学习率、优化器、warmup、权重衰减等默认值。查看命令：

```bash
docker run --rm \
  --network none \
  --read-only \
  --entrypoint sed \
  ywang/yolo-gray1-train:ultralytics-8.3.98-v1 \
  -n '1,320p' \
  /workspace/src/Yolo_fine/ultralytics/ultralytics/cfg/default.yaml
```

### F. 本次训练最终生效参数

训练启动后，合并后的最终参数记录在宿主机：

```text
$RUN_ROOT/experiments/$RUN_ID/args.yaml
```

查看并保存校验和：

```bash
cat "$RUN_ROOT/experiments/$RUN_ID/args.yaml"
sha256sum "$RUN_ROOT/experiments/$RUN_ID/args.yaml"
```

排查某次训练的实际配置时，应首先查看该 `args.yaml`，再依次回查启动命令、项目训练脚本、模型配置、数据集配置和 Ultralytics 默认配置。
