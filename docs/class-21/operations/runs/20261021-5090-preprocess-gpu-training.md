# 2026-10-21：5090 固定原图、GPU 与训练/续训烟测

> 完成时间：2026-10-21 08:44:50（UTC+8）。结果：`PREPROCESS_GPU_TRAIN_RESUME_SMOKE_PASS_WITH_DOCUMENTED_PROFILE`。P2.2、P3.1、P3.2 通过限定范围验收；整体仍为 `IN_PROGRESS`，5/14 步骤通过。训练采用明确记录的 AMP loss scale 适配，不代表默认参数、正式精度或平台全流程均已通过。
> [完成记录](20261021-5090-preprocess-gpu-training/completion.json)、[38 份证据清单](20261021-5090-preprocess-gpu-training/MANIFEST.json)。所有原始失败日志保留，未覆盖此前恢复证据。

## 1. 授权、身份与隔离边界

用户授权依次执行固定原图预处理回归、GPU 运行时验证、训练与续训烟测；10.20 因 SSH 超时未执行，10.21 用户要求重试并在成功后继续。08:34:21 直连 `ssh ubuntu22` 成功，工作根仍为 `/home/ubuntu22/CL01_02`；连接恢复原因未调查，不推断为已修复系统故障。

本轮新输入、脚本、日志与产物统一位于 `records/20261021-smoke/`。H200 只经 `ssh H200` 读取已发布批次的精确文件，不使用 H200 GPU、不写 H200/NAS、不启动服务。没有安装依赖、重建镜像、修改三仓源码、全局 Docker、驱动或网络配置。

三仓固定提交保持[源码身份](../../../agent/topics/REPOSITORIES.md#5090-源码初始化2026-10-20)中的版本，结束时均干净。预处理镜像使用 manifest `17651daceb518e6ebd8220f8bbaa0ddddbfde841d73e7631a1cf66a95d7e8889`，训练镜像使用 `bb5672351a92c50e24a212b6e7405df9a0df214770b18cdfd1d9f70c13c22b53`，仍对应原 H200 配置/层内容。冻结 `models/best.pt` 与参考 ONNX 完成后重新回读，哈希未变；烟测权重不替代业务模型。

CPU 容器无 GPU/网络、4 CPU/8 GiB；GPU 容器按 UUID 绑定唯一 5090，无网络、8 CPU/24 GiB 内存、4 GiB shm、2 GiB tmpfs、UID/GID 1000、只读根文件系统。原数据和模型只读；仅独立记录目录可写。六个测试容器全部退出并保留，失败容器不删除；没有 API/Web 服务部署。

## 2. 固定原图预处理回归

来源为 H200 `/data1/ywang/pose21-raw-gray-test500-20261012-v1/`。执行前按 `source_image` 字典序选择首个地球正样本、月球正样本及空标签样本，冻结[样本计划](20261021-5090-preprocess-gpu-training/fixed-samples/sample-plan.json)，不根据结果换样本。

| 样本 | 原始尺寸 | 类型 |
|---|---|---|
| `agaier_000_view_32` | 4096×4096×3 | 地球正样本 |
| `moon_train_000061` | 2048×2048×3 | 月球正样本 |
| `agaier_495_view_02` | 4096×4096×3 | 空标签 |

仅传输这三张原图及标签、Gray1/net1280/cam2000 参考产物和相关清单，共 30 个文件。原始图/标签匹配 selection 哈希，参考产物匹配各阶段原 SHA256SUMS；传输包为 63,477,760 bytes，SHA-256 `e43e51f74368cc35de7025160bbedba8dcdd5427712d37c87bdc8c058597eddf`。5090 逐文件独立回读后原子发布到 `fixed-samples/`，未恢复整个 500 张工程集或历史原始全集。

实际调用镜像内 `grayprep`，使用固定仓库 profile，显式指定 `r_only`，不采用 cam2000 配置里的通用彩色灰度默认语义。运行前约定像素和 BIN 完全一致，标签格式化文本完全一致，独立几何复算绝对容差 `5.1e-9`、相对容差 0（对应八位小数舍入）。结果见[预处理 JSON](20261021-5090-preprocess-gpu-training/preprocess-result.json)：

- 三张源图均确认 G/B 为 0；R 提取逐像素与 H200 Gray1 参考一致。
- net1280 的 INTER_AREA 缩放至 1000×1000、四边 140 黑边、1280 单通道及标签映射全部一致，并另用 OpenCV/数组公式复算。
- cam2000 从未 padding 的 R 平面独立生成，INTER_LINEAR 至 2000×2000；相机 PNG 逐像素一致。
- 三个 packed-u12 BIN 均为 8,192,000 bytes，与原参考逐字节一致；独立解码验证 4096-byte 行跨度、3000-byte 有效数据、1096-byte 全零行尾和 `round(v*4095/255)`。
- 所有新产物独立回读，输入不改写；此前完整 Gray1 数据的划分/数量/哈希证据仍引用 10.20 恢复记录。

这是三种代表性输入的固定样本回归，不是重跑完整数据生成或 500 张精度评估，也不是 FPGA 实机验收。

## 3. GPU 门禁与运行时

GPU UUID 为 `GPU-90ad73b0-4d19-c3ee-e27f-b4adc6c62f43`。每次 GPU 容器启动前均重新采样 5 次、间隔 2 秒，并读取进程类型、owner 与运行时长；要求无计算进程、各次利用率不高于 20%、空闲至少 24,576 MiB。该值是本次 batch=2 小任务的保守启动门槛，替代无法满足的 H200 32,768 MiB 门槛，不是正式服务配置。

运行中每约 2 秒记录显存/利用率和进程；超 20 分钟、空闲低于 2,048 MiB 或出现不属于本容器的计算进程时，仅停止本任务容器。实际未触发中止；桌面图形进程未停止。启动前空闲约 31,878 MiB，结束时总显存 32,607 MiB、used 209 MiB；驱动保留显存使 used+free 不必等于 total。

[GPU 结果](20261021-5090-preprocess-gpu-training/gpu-result.json)：PyTorch 2.8.0+cu128、torchvision 0.23.0+cu128、本地 Ultralytics 8.3.98；CUDA 12.8、cuDNN 91002，实际 compute capability 为 12.0，架构列表包含 `sm_120`。FP32、FP16、BF16 卷积在 `[2,1,1280,1280]` 上前向/反向均有限；冻结 PT 的 FP16 推理和 warmup 均由 hook 实测为 `[1,1,1280,1280]`。峰值 PyTorch allocated 为 1,159,989,248 bytes。

首版检查直接将 PNG 路径交给通用 `YOLO.predict`，其图像加载路径未保持单通道，触发 shape 断言并停止。按 H200 原烟测的既有契约改为显式读灰度并传入 `[gray[..., None]]` 后通过；只修改独立测试脚本，没有修改 Predictor 或镜像。保留 `gpu.log` 与 `gpu-v2.log`；不得据此宣称通用路径/目录推理入口已经支持 Gray1。速度摘要中的三通道显示也不能覆盖 hook 证据。

本节只验收 PyTorch/CUDA 与 PT 入口，未验收 API/ORT CUDA provider 或重新量化。

## 4. 训练、受控停止与独立续训

从已恢复的 Gray1 数据按 H200 原烟测规则选文件名排序最前的 8 train / 4 val，逐个对照原 `SHA256SUMS` 后复制到独立烟测数据目录，保留原划分和标签。清单见[训练样本](20261021-5090-preprocess-gpu-training/attempt2/training-samples.json)。这 12 张为小型功能集，不能代表 21 类精度。

仍调用镜像内原 `train_yolov8x_p6_pose_gray1.py`，从原 Gray1 YAML 随机初始化，1280、batch=2、workers=2、seed=0、AMP=True、原优化器/增强参数。仅通过独立包装脚本注册输入、梯度、优化器与保存检查，不修改源码或降分辨率。

### 4.1 AMP 适配与失败保留

首轮在默认 GradScaler 初始 scale 65536 下，首层的缩放梯度非有限，被严格检查立即中止；记录 `train.log`，未继续到优化器更新或后续阶段。标准 GradScaler 本身可跳过溢出更新并回退 scale；本次在其处理前停止，因此不能从这次观察断言默认训练或 5090 硬件不可用。

第二轮在新的 `attempt2/` 设置中保持 AMP=True，只在 `on_train_start` 初始化 `GradScaler(init_scale=1.0)`；不放宽损失、梯度或权重有限性检查。四个 batch 的首层梯度均有限，且每次实际优化器更新检查全部参与参数的梯度与权重有限。训练与续训结束 scale 均为 1。这个差异必须随配置记录，不把它描述为原默认参数零差异复现。

原环境的离线 AMP 自检尝试下载 YOLO11n 后跳过，TensorBoard 图形记录仍提示三通道示例不匹配；两者原日志保留。通过结论基于本次真实 Gray1 前向/反向、参数更新与保存/恢复检查，不依赖这些附属自检。

### 4.2 一轮训练及一轮续训结果

预设总计划为 2 epoch，第一进程在完成第 1 个 epoch、验证及保存后用 callback 受控停止。`epoch0.pt` 保留完整优化器状态，避免将最终被 strip 的 `best.pt`/`last.pt` 当作续训源。随后新容器通过原入口 `--resume epoch0.pt` 从 epoch index 1 执行剩余一轮，没有重置训练计数或重新从随机权重开始。

| 项目 | 第一次成功训练 | 独立续训 |
|---|---:|---:|
| 实际训练输入 | `[2,1,1280,1280]` | `[2,1,1280,1280]` |
| start/last epoch（0-based） | 0 / 0 | 1 / 1 |
| 实际优化器更新数 | 3 | 2 |
| EMA 更新计数 | 0 → 3 | 3 → 5 |
| 有限首层梯度检查次数 | 4 | 4 |
| results.csv 累积行数 | 1 | 2 |
| PyTorch 峰值 allocated bytes | 8,378,659,328 | 8,374,050,304 |
| PyTorch 峰值 reserved bytes | 9,506,390,016 | 9,638,510,592 |
| 宿主采样最高 used MiB（非严格峰值） | 9,919 | 10,045 |

续训启动时逐张量比较保存与加载后的优化器状态，1,287 个状态张量按加载 dtype 完全相同；EMA 更新计数正确恢复。两个阶段损失/指标均有限，首层权重确实变化，验证完成，`best.pt` 重载推理成功。随机初始化的小集烟测 mAP 为 0，不作业务精度结论。

证据：[训练结果](20261021-5090-preprocess-gpu-training/attempt2/train-result.json)、[续训结果](20261021-5090-preprocess-gpu-training/attempt2/resume-result.json)、[最终参数](20261021-5090-preprocess-gpu-training/attempt2/training/smoke/args.yaml)、[指标](20261021-5090-preprocess-gpu-training/attempt2/training/smoke/results.csv)。

- `epoch0.pt` SHA-256：`56fa5a1407fe6611ff3254a187114058d9749ea7f8500c197fc8046d70675b67`，续训前后未改变。
- `epoch1.pt` SHA-256：`d0c3bd0f4cff9352ac3be3121ee297569e1d4b00da6186b2859704e0feebfa96`。
- checkpoint 位于 `records/20261021-smoke/attempt2/training/smoke/weights/`，未复制到正式模型目录或 NAS。

本轮证明优化器/EMA/epoch 恢复与继续更新，不证明跨 GPU 逐位复现、任意中途 batch 恢复或完整随机数状态延续。原 checkpoint 不保存 GradScaler 状态；包装脚本在两次启动时显式设为 1，本轮结束 scale 也为 1，不宣称支持一般场景的动态 scaler 无损恢复。

## 5. 执行入口、证据与剩余项

远端记录目录保留 `preprocess_check.py`、`gpu_check_v2.py`、`training_check_v2.py`、`launch_gpu_v3.py` 及所有前版；完整命令、启动门禁和运行采样见证据清单。成功训练/续训入口为 `python3 -B records/20261021-smoke/launch_gpu_v3.py train-v2` 与 `resume-v2`，均需在工作根执行。脚本拒绝覆盖现有容器/任务目录；不能原样重复执行到本次目录，后续须创建新 run ID 并重新门禁。

完成记录独立回读全部 checkpoint、冻结模型与三仓状态。38 份 JSON/JSONL/日志/参数/指标已下载本机并逐文件校验，10 个远端脚本与本机版本哈希一致。证据 TAR SHA-256 为 `241deb5133dd6291eea8200f2bae35da298883f4ec635b502adb3d67850db35f`。打包检查初版因宿主 Python 3.10 没有 `hashlib.file_digest` 失败，保留原脚本后改为分块 SHA-256 并完成，不影响业务测试结果。

完成时根分区剩余 601,762,111,488 bytes，约 560.43 GiB。新增文件与退出容器均保留；无需服务回切，不清理其他任务。本机文档变更前副本位于临时目录 `cl0102-smoke-completion-docs-a4106c6a90ee46218b9001194f5d047b`，原恢复证据/哈希不修改。

下一步先冻结正式数值容差与 H200 对照，再进行 eval462/工程集 PT 测试、ORT CUDA/量化回归与完整八阶段。API/Web、6,600 张全量精度、新环境发布与双向接管仍未执行。通用文件路径推理和默认 loss-scale 的行为边界应继续单列，不用本次有配置约束的烟测覆盖它们。
