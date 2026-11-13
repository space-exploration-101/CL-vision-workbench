# 2026-10-20：5090 运行依赖、挂载与资产恢复

> 完成：2026-10-20 18:41（UTC+8）。本轮状态：`ASSETS_RESTORED_AND_CPU_VERIFIED`；四类镜像、完整 Gray1 train/val/test、标定/评估集及冻结模型已恢复并校验。整体迁移仍为 `IN_PROGRESS`，未验收 GPU、服务或正式精度。[完成记录](20261020-5090-assets-restore/restore-completion.json)、[正式证据清单](20261020-5090-assets-restore/MANIFEST.json)。

## 1. 授权与操作范围

用户在完成 CL01_02 初始化后明确要求“核对运行依赖与挂载，再恢复镜像、数据和模型”。本次在 `ssh ubuntu22` 的 `/home/ubuntu22/CL01_02` 内恢复文件，并向该主机现有 Docker daemon 导入四类项目镜像。

H200 仅经 `ssh H200` 直连读取运行状态、源码和 NAS 已归档资产；访问已有 NAS 路径触发原 autofs，随后核对 CIFS 来源为 `//10.2.26.26/902_data`。没有配置新挂载、向 NAS 写文件、修改 H200 源码、停止其他用户进程或重启服务。5090 未配置 NAS 挂载，恢复数据已落地本机。

CPU 验证使用独立短生命周期容器：显式 `runtime: runc`、无网络、无 GPU 设备、只读根文件系统、UID/GID 1000:1000、4 CPU、12 GiB 内存、2 GiB `/dev/shm`、1 GiB `/tmp`。不启动 API/Web 服务、worker 或 GPU 计算。

## 2. H200 现场基线

2026-10-20 现场读取的四类镜像配置身份与 10.16 归档一致。运行 API 容器 `65c4f5b8f697`、Web 容器 `f60cf6cb7137` 的绑定目录如下；镜像、挂载和宿主源码分别核对，不用 Git HEAD 代替镜像构建身份。

| 服务 | H200 宿主源 | 容器目标 |
|---|---|---|
| API | `/data3/ywang/quantitize-platform/apps/api` | `/home/rs/wrs/onnxviewe/quantitize-platform/apps/api` |
| API | `/data3/ywang/quantitize-platform/pipeline` | `/home/rs/wrs/onnxviewe/quantitize-platform/pipeline` |
| API | `/data3/ywang/quantitize-platform/data/shared_data` | `/home/rs/wrs/onnxviewe/quantitize-platform/data/shared_data` |
| API | `/data3/ywang/quantitize-platform/data/output_data` | `/home/rs/wrs/onnxviewe/quantitize-platform/data/output_data` |
| API | `/data1/ywang/quantitize-scratch` | `/scratch/quantitize` |
| Web | `/data3/ywang/quantitize-platform/apps/web` | `/app` |

API 实际解释器为 `/opt/ywang/yolo-runtime/bin/python`，Python 3.12.12、PyTorch 2.8.0+cu128、torchvision 0.23.0+cu128、ONNX 1.17.0、ORT 1.24.3、NumPy 1.26.4、cv2 4.11.0。实际 Ultralytics 为 8.3.98，来自 `/workspace/src/Yolo_fine/ultralytics/ultralytics/__init__.py`，没有按 pip 元数据升级。

实际 ORT 补丁来自镜像内的平台 `patches/onnxruntime/quantization/`：

| 文件 | SHA-256 |
|---|---|
| `quant_utils.py` | `6afb6b8f1f1dd16a78e46cbed59f389f360c460e5f32c5645dd167cb86bedbe3` |
| `operators/conv.py` | `25898173bffad01e980a5148f3da1718e73a6801f2d391a3081375aaf921b7a6` |

现场验证 QDQ Conv 注册表绑定本地 `QDQConv`，small-scale 辅助函数绑定本地 `compute_scale_zp`。H200 源码中同名补丁字节哈希相同。

两端 `apps/api`、`apps/web`、`pipeline`、`patches` 共 119 个受跟踪文件逐个比较。11 个文件仅 CRLF/LF 换行不同，规范化内容全部一致；保留两端原始 SHA-256 与规范化 SHA-256，未为消除差异而重写源码。证据为 5090 `records/source-comparison.json`。

另将 H200 未跟踪的两个数据选择工具和两个训练诊断材料精确复制到 `assets/h200-supplements/` 并校验；没有将其覆盖进三仓或执行。H200 的控制端路径文档改动不影响运行代码，不复制为 5090 事实。三个 Git 工作区继续使用初始化记录中的固定提交。

现场 API `/health` 和 Web `/health` 均为 200，API `/ready` 仍为 503，未修复。H200 Compose 的 `GPU_MIN_FREE_MIB=32768` 超过 5090 的总显存 32607 MiB；`PT_EVAL_BATCH_SIZE=8`、CUDA 标定 provider 和 150 GiB scratch 空闲门槛也不能作为 5090 已验证配置。本次只准备 CPU 恢复检查，GPU 门槛在 P3 实测后决定，不绕过 readiness。

## 3. 固定资产与传输校验

源归档位置及历史身份见[平台 01](../../../platform/01_NAS备份与环境恢复.md)。本次先验证 10.16 集合 `VERIFIED` 中的清单哈希，再核对根清单与四个资产清单。恢复清单共 70 个文件、45,485,394,119 bytes，另有四份少量补充源码材料。

恢复采用 H200 到本机再到 5090 的 SSH 二进制流，不在本机存放大归档、不复制凭据。目标先写 `.partial`，大小检查和独立回读 SHA-256 通过后同盘原子改名。初始训练镜像传输为提高效率暂停过一次；保留已有原始字节后按偏移续传，改用传输层 gzip 和 AES SSH。最终校验的是未压缩原归档哈希，未改变 NAS 文件。

| 目标（相对 CL01_02） | 内容 | 当前结果 |
|---|---|---|
| `assets/images-20261016/` | 四类镜像、配套源码快照/bundle、原始清单/记录、离线训练运行时 | 已传输；四类镜像内容身份已验收 |
| `assets/data-20260922/` | train/cal462/eval462 TAR 与原始哈希旁文件 | 三个归档与旁文件均通过独立 SHA-256 回读 |
| `datasets/pose21-gray1-1280-m-v1/` | 完整 Gray1 train/val/test、标签与元数据 | 53,597 个普通文件，逐文件回读及原清单通过；无符号链接 |
| `datasets/cali/` | 462 张标定图 | 462 个普通文件，逐文件独立回读通过 |
| `datasets/pose21_gray1_50e_eval462_v2/` | 评估图片、标签与复现材料 | 929 个普通文件，逐文件独立回读通过 |
| `models/best.pt` | 冻结业务 PT | SHA-256 与 MODEL_BASELINE 一致 |
| `models/pose21_gray1_weight_only.onnx` | 从冻结量化 ZIP 中按精确成员路径恢复的参考 ONNX | SHA-256 与 MODEL_BASELINE 一致；ZIP CRC 全部通过 |

PT 为 199,108,106 bytes，SHA-256 `622cff54db472b61c85db03648b36c119f030eacbd4cd87cf5d2dda67e69d6a7`。ONNX 为 100,121,851 bytes，SHA-256 `031def2070a1c2183e62ce5713831bc69cd50dee3a22c10d0ffaff6580d91a02`。未恢复其他历史模型或整套历史任务输出。

数据解包拒绝绝对路径、`..`、符号链接、硬链接及特殊文件，只允许指定顶层目录下的普通文件/目录；逐文件记录归档内容哈希并独立回读落地文件，核对数量后原子发布目录。归档与解包副本同时保留用于恢复对账。

完整 Gray1 目录的普通文件内容合计 15,463,039,349 bytes，原始 `SHA256SUMS` 的 53,594 个条目全部匹配已回读的文件哈希。train/val/test 图像数量分别为 13,560 / 6,634 / 6,600，合计 26,794；各 split 图像与标签的 stem 集合完全相同，未重划分或改写标签。原数据配置保持不变，新配置只改变 `path`，21 类、单通道和关键点语义均经容器内比对通过。

## 4. 两类 Docker 身份的对应

H200 使用 `overlay2`，5090 返回的驱动状态为 `io.containerd.snapshotter.v1`。现场发现 5090 `.Id` 对应原归档的 OCI manifest digest，而 H200 `.Id` 对应原配置 blob digest。若只比较显示的 `.Id` 会误报不一致。

本次逐项验证：原 TAR 哈希 → `manifest.json` 的配置内容 SHA-256 → `index.json` 引用的 OCI manifest 字节 SHA-256 → manifest 引用的配置 digest → 5090 `.Id` → 全部 RootFS diff IDs 与运行配置字段。只允许默认空 `User`/`WorkingDir` 的 `""` 与 `null` 表示差异，不允许替换其他已有镜像标签。

| 镜像 | H200 配置 ID | 5090 manifest ID |
|---|---|---|
| 预处理 | `sha256:ae238a8a9099ef31a6c06ef788d964ec9dcc4875f80958cd765c1614b90892db` | `sha256:17651daceb518e6ebd8220f8bbaa0ddddbfde841d73e7631a1cf66a95d7e8889` |
| 训练/PT | `sha256:79b0086e39b83c8a3bbb76bfe2df1d8683bd443a6bef19ee7d2e69f096033635` | `sha256:bb5672351a92c50e24a212b6e7405df9a0df214770b18cdfd1d9f70c13c22b53` |
| API | `sha256:f633b01c2f85d1fd39bf01e9b9f556056d03c805529781c8879e3172412e8ed2` | `sha256:28f60e0c4ae9b2f94634685a7fb3c00ccbf3fcc7f98535fa170a4e6aab9288fd` |
| Web | `sha256:7aef48747d3fc4270f85133382ffbcf83ea0c53e831f06978ac777b48743d9e9` | `sha256:0b38dfdbac14613c262ef5277af69ccbc2107782dcd186eca52e9c1b686c1599` |

四个 `records/<资产名>-image-verified.json` 保存验证结果及全部层 diff IDs。镜像内容相同，没有升级任何算法依赖；宿主驱动、Docker 存储后端不同，不承诺宿主环境完全相同。

## 5. 5090 隔离挂载与 CPU 验证

部署检查文件为 `/home/ubuntu22/CL01_02/deployment/compose.cpu-check.yml`，按上表不可变 manifest 固定镜像。它只有 `train-check`、`api-check`、`preprocess-check`、`web-check` 四个一次性检查入口，不是 API/Web 服务启动配置，不发布端口。

| CL01_02 下的源 | 容器用途 |
|---|---|
| `quantitize-platform/apps/api`、`pipeline`、`patches` | 只读挂载到原 API 内部平台路径；补丁与 H200 实际使用内容一致 |
| `quantitize-platform/apps/web` | 只读 `/app` |
| `datasets/`、`deployment/gray1.dataset.yaml` | 只读 `/datasets` 和 `/configs/gray1.dataset.yaml`；保留原数据集配置，另生成目标路径配置 |
| `datasets/cali`、`datasets/pose21_gray1_50e_eval462_v2` | 只读挂载到 API 原共享数据逻辑路径，避免断链 |
| `models/` | 只读 `/models` |
| `output_data/`、`scratch/` | 本机新建独立输出与临时目录；不复制 H200 队列 |
| `runs/`、`experiments/` | 本机新建训练结果目录；未启动训练 |

已通过的 CPU 检查：

- 预处理：Grayprep 0.2.1、NumPy 1.26.4、cv2 4.11.0，实际样本 `aipuha_000_view_31.png` 为 `(1280,1280)` uint8 单通道。
- Web：Python 3.11.16、FastAPI 0.128.8、Uvicorn 0.39.0，宿主源码挂载可读。
- 训练/PT：Python 3.12.12、PyTorch 2.8.0+cu128、本地 Ultralytics 8.3.98；冻结 PT 加载成功，首层单通道、21 类、关键点 `[2,3]`。
- API/ORT：本地补丁哈希与 H200 相同，注册表和辅助函数绑定通过；ONNX 图为 148 个 Conv/148 个权重 DQ、无 QuantizeLinear，输入 FP16 `[1,1,1280,1280]`。

API 检查还使用标定图 `agaier_047_view_18.png` 做一次 CPU 前向。PT 约 2.92 秒，ONNX 约 2.68 秒，实际 provider 为 `CPUExecutionProvider`；两者输出均为 `[1,31,34000]` 且数值有限。这里只验收模型可加载和执行，未计算或验收精度差，也没有用这次时长作 GPU 性能结论。

首版预处理验证脚本误用 Pillow，原镜像并未安装该包；保留失败记录，改为使用项目锁定的 OpenCV 后通过，没有为测试安装新依赖。镜像导入检查先遇到 manifest/config ID 差异，随后遇到训练镜像缺省 User 的 `""`/`null` 差异；均先读取证据再修正验证器，保留脚本前版。没有放宽内容哈希、层或模型身份标准。

## 6. 证据、回滚与剩余项

5090 证据目录为 `/home/ubuntu22/CL01_02/records/`；数据成员清单、解包验证、模型身份、镜像内容验证、源码比较和 CPU 检查 JSON 均在此。任务脚本位于同目录并保留前版。本机已独立回读并按 SHA-256 核对 18 份正式 JSON，完整文件列表见[证据清单](20261020-5090-assets-restore/MANIFEST.json)。本机 `.tmp/cl0102-restore/` 是临时传输与验证工作区，不是当前状态来源。

最终复核 70 个归档/模型资产，合计 45,485,394,119 bytes；四个原资产清单和集合清单全部通过。`assets/` 与 `datasets/` 无 `.partial`，检查容器全部退出，三个源码工作区均干净。`records/` 中保留早期失败验证的临时记录和旧脚本，不把它们当作已发布资产。完成时根分区可用 610,250,608,640 bytes（约 568.34 GiB）；HOME 与 Docker 仍共用该分区，未迁移 Docker 全局存储。

本次没有切换任何现有服务，因此不需要回切生产入口。新增镜像、归档、数据和模型原位保留用于审计；如后续撤销，先核对被引用的镜像/目录，再单独决定清理，不删除其他项目镜像或旧 NAS 发布。

本轮请求范围已完成，P0.1 与 P2.1 可标记通过。未恢复约 836 GiB 的历史原始全集、500 张工程批次及全部历史任务输出；本次数据范围为上述 Gray1 三套划分、cal462 和 eval462。后续完整预处理回归需按精确清单取得固定原图/标签与参考产物。

GPU 算子/训练、正式精度与重新量化八阶段、API/Web 服务、双向接管仍未执行；H200 API ready 503 与 5090 GPU 门槛仍需处理。单张 CPU 前向及资产完整不能替代这些验收。
