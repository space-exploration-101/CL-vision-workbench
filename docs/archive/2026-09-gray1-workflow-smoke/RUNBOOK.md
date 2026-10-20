# 工作流运行手册

本文档的精确数据路径和hash以`EXECUTION_RECORD.md`和`DATASET_INVENTORY.json`为准。

## 1. 数据准备

数据生成与grayprep处理已由本次执行完成：

```bash
DATASET=/data1/ywang/workflow-test/synthetic-pose21-r-only-v1
test -f "$DATASET/VERIFIED"
```

重新执行时必须使用新的目标版本目录，禁止覆盖已有目录。

```text
synthetic source (R-only uint8 PNG + YOLO pose labels)
  -> deterministic selection
  -> net1280 gray1 images/labels
  -> cam2000 packed BIN for test split
  -> strict verify
```

推荐grayprep参数：`--workers 32 --opencv-threads 1`。源目录只读，输出使用`.partial`和原子发布。

## 2. 训练烟测

训练环境：`/data3/ywang/yolo-gray1-train`。

镜像：`ywang/yolo-gray1-train:ultralytics-8.3.98-v1`。当前gray1入口从模型配置初始化，不加载三通道预训练权重。

训练前复制prepared dataset的`dataset.yaml`到任务目录，并将`path`改为容器内只读挂载`/dataset`。GPU使用前必须连续检查利用率、显存、进程owner和运行时间；烟测限制为单卡、1 epoch、预计小于1小时。

建立独立运行目录，并修正容器内dataset根路径：

```bash
RUN_ID="e2e_smoke_$(date +%Y%m%d_%H%M%S)"
DATASET=/data1/ywang/workflow-test/synthetic-pose21-r-only-v1/prepared
RUN_ROOT="/data1/ywang/workflow-test/runs/$RUN_ID"
mkdir -p "$RUN_ROOT/experiments"
cp "$DATASET/dataset.yaml" "$RUN_ROOT/dataset.container.yaml"
sed -i 's|^path:.*|path: /dataset|' "$RUN_ROOT/dataset.container.yaml"
```

训练前连续检查GPU及进程owner；不要根据一次`0%`判断空闲：

```bash
nvidia-smi --query-gpu=index,uuid,name,memory.used,memory.free,utilization.gpu --format=csv
nvidia-smi dmon -s pucm -c 5
nvidia-smi --query-compute-apps=gpu_uuid,pid,process_name,used_memory --format=csv
```

选择经协调的单卡UUID后运行1 epoch。该结果只证明工作流，不证明精度：

```bash
GPU_UUID='<approved-idle-gpu-uuid>'

docker run --rm \
  --gpus "device=$GPU_UUID" \
  --user 1012:1012 \
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
  --name "$RUN_ID"
```

预期模型：

```bash
MODEL="$RUN_ROOT/experiments/$RUN_ID/weights/best.pt"
test -s "$MODEL"
sha256sum "$MODEL"
```

## 3. 量化烟测

上传训练产生的`best.pt`。不得选择现有shared datasets；为保证无外部数据，必须从本数据集组装ZIP：

```bash
QPKG="$RUN_ROOT/quant-input"
mkdir -p "$QPKG/input/cali" "$QPKG/input/test/images" "$QPKG/input/test/labels"
cp "$MODEL" "$QPKG/input/model.pt"
cp "$DATASET"/images/calibration/*.png "$QPKG/input/cali/"
cp "$DATASET"/images/test/*.png "$QPKG/input/test/images/"
cp "$DATASET"/labels/test/*.txt "$QPKG/input/test/labels/"
```

在`$QPKG/job_config.json`写入：

```json
{
  "nc": 21,
  "imgsz": 1280,
  "min_test_images": 1,
  "min_cali_images": 1,
  "preprocess_mode": "passthrough",
  "input_channels": 1,
  "input_semantics": "gray1",
  "use_scratch": true
}
```

打包：

```bash
(cd "$QPKG" && zip -qr "$RUN_ROOT/quant-input.zip" input job_config.json)
```

对于当前单通道数据使用：

```yaml
preprocess_mode: passthrough
imgsz: 1280
nc: 21
input_channels: 1
input_semantics: gray1
```

量化前`/health`和`/ready`必须同时成功：

```bash
curl -fsS http://127.0.0.1:8000/health
curl -fsS http://127.0.0.1:8000/ready
```

当前`/ready`为503时不得启动或绕过门禁。恢复ready后创建任务：

```bash
curl -fsS -X POST \
  -F "display_name=$RUN_ID" \
  -F "onnx_name=$RUN_ID" \
  -F "preprocess_mode=passthrough" \
  -F "archive=@$RUN_ROOT/quant-input.zip" \
  http://127.0.0.1:8000/api/tasks
```

取得返回的`task_id`后启动并查询：

```bash
TASK_ID='<returned-task-id>'
curl -fsS -X POST "http://127.0.0.1:8000/api/tasks/$TASK_ID/start"
curl -fsS "http://127.0.0.1:8000/api/tasks/$TASK_ID" | python3 -m json.tool
```

当前量化平台可能在`generate_bin`失败；必须保留真实失败状态，不伪造merge或bundle成功。新框架应将与登记指纹一致的结果表示为`known_blocked`，后续步骤为`skipped_dependency`，整体最多为`software_validated_partial`。
