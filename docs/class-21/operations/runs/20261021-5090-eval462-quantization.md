# 2026-10-21：5090 eval462 回归与重新量化

## 1. 授权与边界

用户在确认数据范围后要求“接着运行吧”。本轮只使用固定单图 `agaier_047_view_18.png`、eval462 和 cal462，以及冻结业务 PT/参考 ONNX；不使用训练烟测权重，不运行完整 500 张工程集或 6,600 张 test，不启动八阶段打包、API/Web 或 NAS 发布。H200 只读获取原任务对照，不使用 H200 GPU。

同日用户进一步明确移除 P4.4 全量回归：6,600 张测试不再是本迁移待办或环境完成门禁，当前 462 张任务不受影响。原始 `comparison-plan.json` 本来就排除了该全量测试，因此保持原文件不改。

5090 通过 `ssh ubuntu22` 连接；独立任务目录为 `/home/ubuntu22/CL01_02/records/20261021-eval462`。原数据、模型与源码以只读方式挂入检查容器；新导出写入该目录的 `candidate/`，不替换 `/home/ubuntu22/CL01_02/models/` 内的冻结资产。大模型先写 `.partial`，结构/数值校验后原子改名。当前记录为执行中，最终证据清单在收尾后补充。

## 2. 固定身份与比较口径

- PT SHA256：`622cff54db472b61c85db03648b36c119f030eacbd4cd87cf5d2dda67e69d6a7`；参考 ONNX SHA256：`031def2070a1c2183e62ce5713831bc69cd50dee3a22c10d0ffaff6580d91a02`。
- 本轮 PT 评估、导出与量化统一使用已恢复的 API 镜像 `quantitize-platform-api@sha256:28f60e0c4ae9b2f94634685a7fb3c00ccbf3fcc7f98535fa170a4e6aab9288fd`，不是重新打包的训练镜像。前一轮训练/续训使用的训练镜像见[独立记录](20261021-5090-preprocess-gpu-training.md)。
- 解释器 `/opt/ywang/yolo-runtime/bin/python`，本地 Gray1 从 `/workspace/src/Yolo_fine/ultralytics` 导入；Python 3.12.12、PyTorch 2.8.0+cu128、Ultralytics 8.3.98、ONNX 1.17.0、ORT 1.24.3。未安装或升级依赖；导出器曾提示缺少 `onnxruntime-gpu` 并尝试自动更新，但容器无网络，日志确认跳过更新，导出仍成功。
- 平台固定提交 `b1ac5ea84e0374d9bfa2be2c9214e07e950a0a79`；挂载其 `apps/api`、`pipeline`、`patches`。沿用原 `eval_common` 的匹配/汇总和原 `quantitize` 的导出/量化函数，独立包装只控制输入、设备、资源、观测和门禁。
- H200 对照来自 `/data3/ywang/quantitize-platform/data/output_data/20260920_063903_Pose21_Gray1_weight-only_accuracy_valida`。读取时按已验证宿主挂载映射解析历史容器绝对链接，未更改链接。下载的原配置、输入校验和 PT/ONNX 结果/日志共 13 份逐项校验；归档 SHA256 为 `db7050cc043a61bb3518232da04ab28cc741613d8b2a0a76a1ab4fb744764cfe`。
- 5090 的 cal462 图片、eval462 图片与标签各 462 份，共 1,386 项 SHA256 与 H200 原任务实际输入逐项一致。cal/eval 文件名无交集；eval462 为 378 张正样本和 84 张空样本，不是正式全量 test。

运行前于 09:41:11（UTC+8）冻结 `comparison-plan.json`：单通道 1280、passthrough、conf=0.25、NMS IoU=0.7、同类 IoU≥0.5 匹配、max_det=300、nc=21；PT 为 CUDA FP32、batch=2，ORT 评估显式 CPU、intra/inter threads=4/1。不以通用文件路径调用 Gray1 predictor，使用显式 HWC1 数组，并通过首层 hook 检查实际张量。首轮校准函数另建的 session 未继承评估线程配置；下述重试固定为同样的 4/1，数据/数值门禁不变。

迁移回归门禁：precision/recall/F1/任务 mAP 各自绝对差≤0.005；平均框中心误差差≤0.5 像素；总 TP/FP/FN 各差≤2，逐类各差≤1。固定单图要求检测类别/数量一致、匹配框坐标最大差≤5 像素、置信度差≤0.05。跨 FP32 与量化 FP16 的原始张量最大/平均误差另作诊断，不使用逐位相同要求。这里的任务 mAP 是逐类 precision×recall 的平均，不是 COCO mAP；这些是本轮工程迁移门禁，不是新增业务精度承诺或完整关键点验收。

## 3. 实际运行路径

### ORT provider 与校准算法

5090 原 API 镜像的 `ort.get_available_providers()` 只有 `AzureExecutionProvider` 和 `CPUExecutionProvider`，没有 CUDA。H200 原 ONNX 评估日志也明确警告 CUDA 不可用，最终实际 session 为 CPU。因此本轮显式选择 CPU 并禁用 session fallback，不把 CPU 结果当作 ORT CUDA 通过。

H200 原量化日志中的“CUDAExecutionProvider, CPUExecutionProvider”只是代码打印的请求列表，不能证明实际使用 CUDA。本轮在原量化 factory 外增加只观测包装，记录实际校准 session 为 `CPUExecutionProvider`、校准器类型为 `MinMaxCalibrater`。原函数虽含 `CalibrateMethod: Entropy` 的 extra option 和相关日志字样，但没有传递真正的 `calibrate_method` 参数；本轮保留原实现，不擅自改为 Entropy。

本地 `quant_utils.py`、`operators/conv.py` 的 SHA256 分别为 `6afb6b8f1f1dd16a78e46cbed59f389f360c460e5f32c5645dd167cb86bedbe3`、`25898173bffad01e980a5148f3da1718e73a6801f2d391a3081375aaf921b7a6`；实际模块路径、Conv 注册表与 small-scale 函数绑定均验证通过。

### 已完成结果与执行中项

截至 11:06（UTC+8）：

| 检查 | 结果 | 边界 |
|---|---|---|
| 输入身份与 PT 单图 | 通过；固定 PT，实际 `[B,1,1280,1280]` CUDA FP32 | 不使用烟测权重 |
| PT eval462 | 通过；TP/FP/FN=331/31/47，21 类计数及汇总指标与 H200 相同；平均框中心误差差约 0.002131 像素 | 不是完整 500/6,600 张或关键点正式验收 |
| 参考 ONNX 单图 | 相对 PT 的类别/数量一致；框最大差 1.430115 像素、置信度差 0.000219；原始张量 max/mean≈4.337524/0.041007 | 实际 CPU，不是 ORT CUDA |
| 参考 ONNX eval462 | 通过；TP/FP/FN=331/31/47，21 类计数及汇总指标与 H200 相同；平均框中心误差差约 0.001427 像素 | CPU 受限配置，整阶段约 977 秒 |
| 新 FP16 导出 | 通过；静态 `[1,1,1280,1280]`、FP16、opset17、onnxslim 0.1.87；单图相对 PT 框最大差 0.930115 像素 | 从冻结 PT 的独立可写副本导出，原权重不变 |
| cal462 重新量化 | 四线程重试通过；实际消费 462/462 张，148 Conv/148 权重 DQ、逐通道 QInt8、FP16 输入、无激活 QDQ；整阶段约 1,040 秒 | 首轮默认线程尝试受控停止并保留；结构通过不代替新模型精度回归 |
| 新量化模型 eval462 | 执行中 | 结果收齐前不标记 P4.2 完成 |

### 校准线程适配与保留的首轮尝试

首轮 `quantize` 使用原校准器默认 SessionOptions。现场观察 Python 进程共 30 个线程，容器仅配额 8 CPU；cgroup 几乎每个周期均发生节流。运行约 2,861.66 秒仍未结束，于 10:47 左右仅停止本任务的 `cl0102-eval462-quantize-20261021`，退出码 137，原容器和日志保留。该退出来自受控 stop，不是数值门禁失败；是否 OOM 以最终容器状态证据确认，不仅凭退出码判断。

`quantize-v2` 在独立包装内临时替换校准器的 session 创建方法，固定 intra/inter threads=4/1，仍保留原 `ORT_DISABLE_ALL`、CPU provider、MinMax、462 张顺序、排除节点和 weight-only 参数。增加逐图读取计数/有限性检查，未改平台仓库、镜像或依赖。462 张全部消费、量化结构校验通过后，已将 `candidate/weight_only-v2.onnx.partial` 原子改名为候选 `weight_only.onnx`。原尝试容器 `OOMKilled=false`，退出码 137 是受控停止结果。后续复现需要保留 `quantize_retry.py` 的线程适配，不能将本次重试说成默认线程零差异运行。

## 4. 资源、证据与未完成项

GPU 阶段仅 PT 和导出；每次开始前 5 次间隔 2 秒采样利用率、显存和进程 owner，要求至少 24,576 MiB 空闲、利用率≤20%、无外部计算进程。运行中每 5 秒监控；如剩余显存低于 2,048 MiB、出现外部计算竞争或超过 3,600 秒，只停止本任务容器。桌面进程不停止。CPU 阶段不暴露 GPU。

检查容器统一无网络、只读根文件系统、UID/GID=1000、8 CPU/32 GiB 内存限额、shm=4 GiB、临时目录 3 GiB。参考评估实际内存观察约 9.47 GiB，量化约 8.03 GiB；这些是观测点，不是完整峰值或正式服务容量承诺。

执行入口为任务目录的 `launch.py` 与 `workflow.py`，各阶段保存 `*-command.json`、`*-runtime.json`、`*-preflight.json`、`*-monitor.jsonl`、`*-exit.json` 和日志。对照结果在 `results/`，独立模型在 `candidate/`，H200 原始对照在 `h200-baseline/`。重新运行须新建任务目录，不覆盖本轮证据。

未完成：ORT CUDA、完整 500 张工程集、关键点标准、八阶段/逐层/BIN、API/Web readiness、恢复交付和双向接管。6,600 张全量回归按用户要求移出本次范围，不作为阻塞，也不宣称已通过。当前总体仍为 `IN_PROGRESS`；逐项进度只在[迁移计划](../../../platform/03_5090训练测试推理与量化运行环境.md)维护。
