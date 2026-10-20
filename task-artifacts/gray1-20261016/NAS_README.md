# YOLO Gray1 1280 训练与推理镜像备份

日期：2026-10-16。备份状态以本目录 `VERIFIED` 和 `SHA256SUMS` 为准。

## 镜像与源码身份

- 镜像：`yolo-gray1:1280-train-predict-20261016`，Linux x86_64。
- 镜像 ID：`sha256:79b0086e39b83c8a3bbb76bfe2df1d8683bd443a6bef19ee7d2e69f096033635`。
- 功能：1280×1280 单通道灰度图训练、验证与 PT 推理，21 类、每目标 2 个关键点。
- GitHub：`https://github.com/space-exploration-101/yolo-gray1-train`。
- 分支：`publish/h200-gray1-20261015`。
- 镜像构建源码：`5b3caca445874b3567c0ad0b425c02c8d6c920e2`。
- 发布说明与默认入口：`c981f5b8a7a4f3c8a111c63812d0c74f5fe8b5f2`。Dockerfile 和训练推理源码与镜像构建版本一致，发布提交不代表再次重建镜像。

## 文件清单

| 文件 | 用途 |
|---|---|
| `yolo-gray1_1280-train-predict-20261016.tar` | `docker save` 的未压缩完整镜像，使用 `docker load` 导入 |
| `yolo-runtime.tar.gz` | 完整 Dockerfile 重建所需离线 Python/CUDA 用户态运行时 |
| `source.bundle` | 发布分支完整 Git 历史，可用于离线 clone |
| `source.tar.gz` | 发布提交的受跟踪源码快照；从仓库根目录打包，无外层目录 |
| `records/source-state.json` | 构建与发布提交、远端 refs、未纳入发布的未知文件名 |
| `Dockerfile`、`BUILD_IMAGE.md` | 与已验收镜像相同的完整构建定义，以及依赖说明 |
| `records/image-inspect.json`、`records/archive-verification.json` | 镜像身份、各层摘要、TAR 与依赖校验、NAS 导入验证 |
| `records/obsolete-images-before.json` | 清理前旧镜像身份记录 |
| `records/smoke/` | 原始训练与 CPU/GPU 推理日志、验证脚本、NAS 导入后 CPU 复测记录 |
| `records/cleanup.json` | 三个旧 train/predict 标签的实际清理结果；不存在时表示尚未完成清理 |
| `SHA256SUMS`、`VERIFIED` | 文件哈希与备份完成标记 |

校验明细和烟测材料统一归档在 `records/`，根目录只保留使用、恢复和构建所需材料，以及便于直接验收的 `SHA256SUMS`、`VERIFIED`。从备份根目录执行 `sha256sum -c SHA256SUMS` 的方式不变。源码快照及 Git bundle 保持原发布版本，其内文档反映原始发布布局；当前 NAS 文件位置以本 README 为准。

镜像 TAR SHA-256：`ae6a9919c3204990af34f06918739d3ea97454560d43962365c1c14243d0ccb1`。

运行时包 SHA-256：`e1e2f5eb38d40ab81c8075ef45624da04d24420e42807f78c5e9f8903e8af683`。

## 推荐恢复方式：直接导入完整镜像

在已授权的 Linux Docker 主机上，确认 NAS 实际挂载为 `//10.2.26.26/902_data`、CIFS，然后执行：

```bash
set -euo pipefail
NAS_ROOT='/mnt/ywang-nas/0-项目/13-专项/4-代码/20261016_H200_docker_images/yolo_gray1_1280_train_predict'
findmnt -t cifs -T "$NAS_ROOT" -o TARGET,SOURCE,FSTYPE
test -r "$NAS_ROOT/VERIFIED"
(cd "$NAS_ROOT" && sha256sum -c SHA256SUMS)
IMAGE='yolo-gray1:1280-train-predict-20261016'
EXPECTED='sha256:79b0086e39b83c8a3bbb76bfe2df1d8683bd443a6bef19ee7d2e69f096033635'
if docker image inspect "$IMAGE" >/dev/null 2>&1; then
  test "$(docker image inspect "$IMAGE" --format '{{.Id}}')" = "$EXPECTED"
fi
docker load --input "$NAS_ROOT/yolo-gray1_1280-train-predict-20261016.tar"
test "$(docker image inspect "$IMAGE" --format '{{.Id}}')" = "$EXPECTED"
docker run --rm --network none --read-only --tmpfs /tmp:rw,size=1g \
  -e HOME=/tmp -e NO_ALBUMENTATIONS_UPDATE=1 "$IMAGE" \
  python /workspace/src/Yolo_fine/train_yolov8x_p6_pose_gray1.py --help
```

镜像内包含完整运行时和训练/推理源码，运行时无需挂载本离线包、宿主 Python 或源码。训练数据、模型权重、输出目录需单独提供。GPU 调用还依赖宿主 NVIDIA 驱动与 Container Toolkit；使用前检查 GPU 利用率、显存、进程所有者并持续采样。

## 从 Dockerfile 重建

构建需要：本目录运行时包、`source.tar.gz` 或发布分支的完整源码、`docker/Dockerfile`、`.dockerignore` 和 `ubuntu:22.04` 基础镜像。基础镜像未单独导出到本目录；离线重建前需自行准备。直接导入完整镜像不需要单独准备基础镜像。

恢复到新建空目录并保留权限：

```bash
mkdir yolo-gray1-build
tar -xzf "$NAS_ROOT/source.tar.gz" -C yolo-gray1-build
cd yolo-gray1-build
cp "$NAS_ROOT/yolo-runtime.tar.gz" docker/yolo-runtime.tar.gz.partial
printf '%s  %s\n' \
  e1e2f5eb38d40ab81c8075ef45624da04d24420e42807f78c5e9f8903e8af683 \
  docker/yolo-runtime.tar.gz.partial | sha256sum -c -
gzip -t docker/yolo-runtime.tar.gz.partial
mv docker/yolo-runtime.tar.gz.partial docker/yolo-runtime.tar.gz
docker image inspect ubuntu:22.04 >/dev/null
docker build --network=none --pull=false -f docker/Dockerfile \
  -t yolo-gray1:1280-train-predict-candidate .
```

基础镜像原 ID 为 `sha256:714d1a2f39e9830a7c0453dc924ddf849be74811c13e4afaa21c0b9b6ad74af0`，Dockerfile 中标签可变，重建不保证产生相同 ID。离线包包含 Python 3.12.12、PyTorch 2.8.0+cu128、torchvision 0.23.0+cu128、NumPy 1.26.4 等；依赖记录不能代替离线包。构建不需要旧 v1 镜像或三通道预训练权重。完整细节见 `BUILD_IMAGE.md`。

## 验证与清理边界

原镜像已通过 GPU 训练烟测：8 张 train、4 张 val、batch=2、1 epoch，实际输入 `[2,1,1280,1280]`，保存 checkpoint。CPU/GPU 推理实际输入和 warmup 均为 `[1,1,1280,1280]`，GPU 峰值分配 7,577,967,104 bytes。只用于功能验证，随机初始化的一轮训练结果不能作为业务模型或精度验收。

TAR 配置哈希与完整镜像 ID 一致，各层 diff ID、运行时 gzip、NAS 大文件独立回读 SHA-256 均检查；NAS 导入后核对 ID 并执行 CPU 推理，日志见 `records/smoke/nas-predict-cpu.log`。验证在现有 H200 daemon 上进行，不能称为新主机 clean-room 恢复。

完成上述备份验收后，仅清理无容器引用的历史训练 `ultralytics-8.3.98-v1`、`ultralytics-8.3.98-v2` 和 `ultralytics-8.3.98-v1-predictor-gray1` 三个标签；实际 ID 删除情况见 `records/cleanup.json`。不强制删除、不执行全局 prune，不修改 API/Web、预处理或其他用户镜像。历史源码中的增量 Dockerfile 仍可回溯，但它依赖旧 v1，当前使用完整 Dockerfile。

备份不含正式数据集、正式模型权重、API/Web 或 FPGA 实机资产；整套平台恢复状态仍为 `PARTIAL`。

本次旧镜像清理前观察到：API/Web `/health` 均 HTTP 200，但 API `/ready` 返回 HTTP 503，原因是 API 容器内 NVML 初始化失败（`Failed to initialize NVML: Unknown Error`）。宿主 GPU 查询正常，本次新训练镜像 GPU 烟测已通过；未重启或修复 API 服务，检查记录见 `records/smoke/service-checks.json`。
