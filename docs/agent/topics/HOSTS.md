# 主机、服务和路径

依据既有记录整理于 2026-10-18；2026-10-20 已核对 Windows 工作区更名，见[完成记录](../../archive/2026-10-workbench-rename/RENAME_COMPLETION.md)。可变状态操作前现场核对。

| 项目 | 已记录信息 |
|---|---|
| Windows | 本机工作区为 `E:\codex\CL-vision-workbench`；`E:\codex\quantitize` 为旧根路径。根 Git 管理文档、数据与日志，脚本及临时材料仅本机保留；旧 H200 本机源码镜像已退役 |
| H200 | `10.2.29.180`，用户 `ywang`；必须 `ssh H200` 直连，禁止 wrs 跳板 |
| 历史硬件记录 | Ubuntu 22.04.5，128 逻辑 CPU、约 1 TiB 内存、4×H200 NVL |
| 平台源码 | `/data3/ywang/quantitize-platform`，现场读取 HEAD、status 和 remote |
| 预处理/训练仓库 | `/data3/ywang/yolo-gray1-data-pipeline`、`/data3/ywang/yolo-gray1-train` |
| Compose | 平台目录内 `deploy/docker-compose.yml`、`deploy/.env`；禁止输出完整凭据文件 |
| Web/API | `http://10.2.29.180:8088` / `http://10.2.29.180:8000` |
| 健康检查 | API `/health`、`/ready`，Web `/health` |
| 输入/输出 | 平台目录下 `data/shared_data` / `data/output_data/<task_id>` |
| 临时 workspace | `/data1/ywang/quantitize-scratch/<task_id>`，以 JobConfig/manifest 为准 |
| NAS | `//10.2.26.26/902_data`，历史挂载点 `/mnt/ywang-nas`，10.16 记录由 autofs 触发 CIFS 挂载；当前挂载方式须现场复核 |

Windows 路由历史值为 `10.2.29.0/24`、`10.2.26.0/24` 经 `192.168.31.1`、ifIndex 19；连接排障先查当前路由。GPU UUID `GPU-1610284d-3afe-6e14-b8cb-bbbe40a1f28b` 曾用于任务，不能视为长期独占。历史 readiness 门槛为 32 GiB 空闲、利用率不高于 20%，现场以部署配置和任务峰值为准。


最新源码位置另见 [REPOSITORIES](REPOSITORIES.md)。诊断命令见 [DIAGNOSTICS](../tasks/DIAGNOSTICS.md)；不能从表中历史配置判断 GPU 当前空闲或服务就绪。

## 5090：2026-10-20 现场核对

基础盘点见[10.20 初始化](../../class-21/operations/runs/20261020-5090-cl01-02-bootstrap.md)与[资产恢复](../../class-21/operations/runs/20261020-5090-assets-restore.md)；10.21 已新增[固定原图、GPU 与训练/续训烟测](../../class-21/operations/runs/20261021-5090-preprocess-gpu-training.md)，未部署服务。

最新连接观察：10.21 08:34:21（UTC+8）原入口直连成功，随后完成烟测。此前[10.20 超时](../../class-21/operations/runs/20261020-5090-smoke-connectivity-blocked.md)原因未查明，未修改网络配置。硬件基础信息沿用 10.20，容量/GPU 观察按下表明确时点，不视为长期资源预留。

| 项目 | 已核对信息 |
|---|---|
| SSH | `ssh ubuntu22`，使用小写已有别名；`ubuntu22@10.2.26.79:22`，登录成功 |
| 系统与账户 | Ubuntu 22.04.5 LTS、x86_64；用户 `ubuntu22`，UID/GID 1000:1000；32 逻辑 CPU、约 62.6 GiB 物理内存 |
| 工作根目录 | `/home/ubuntu22/CL01_02`，已创建；三个仓库直接位于其下，版本见 REPOSITORIES |
| 存储 | HOME 与 Docker root `/var/lib/docker` 共用 `/dev/nvme1n1p3` ext4 根分区；10.21 08:44:50（UTC+8）烟测完成时可用 601,762,111,488 bytes（约 560.43 GiB），不能将两处容量相加 |
| GPU | RTX 5090；UUID `GPU-90ad73b0-4d19-c3ee-e27f-b4adc6c62f43`；32,607 MiB；驱动 580.178.04 |
| GPU 占用观察 | 10.21 每次启动前 5 次采样并查 owner，无外部计算任务才启动；结束时利用率 0%、used 209 MiB/free 31,878 MiB；桌面进程未停止，后续使用前重新采样 |
| 容器基础软件 | 10.20 Docker 29.7.2、Compose v5.5.0、NVIDIA Container Toolkit/libnvidia-container1 1.20.0-1；10.21 原镜像 PyTorch CUDA 算子及适配配置的训练/续训通过，ORT CUDA 未验收 |
| 网络与部署缺口 | GitHub SSH 22 端口连接在密钥交换阶段被重置，鉴权未验证；归档经 H200 既有 NAS 读取通路落地，5090 未挂载 NAS；CPU Compose 和独立 GPU 烟测配置已验证，正式服务入口/鉴权与 GPU readiness 未验收 |

当前已写入三仓、原始归档、完整 Gray1 train/val/test、cal462/eval462、冻结模型、独立配置与验证记录，并导入四类镜像；全部任务峰值仍需预算。约 836 GiB 的历史原始全集超过现有空闲容量，不在本次恢复范围。HOME 与 Docker 共用盘的设置未改动，未部署服务或写入 NAS。
