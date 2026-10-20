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
