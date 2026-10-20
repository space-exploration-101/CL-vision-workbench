# 重建材料规则

继承根 [AGENTS](../AGENTS.md)。先读 [README](README.md) 与 [CURRENT_STATE](CURRENT_STATE.md)，需要执行时再读 [REBUILD_GUIDE](REBUILD_GUIDE.md)。

- 当前身份详细来源为[平台说明](../docs/platform/README.md)中的 01/02；不要从 legacy 的旧 Python、CUDA 路径或镜像标签推导当前运行方式。
- legacy 脚本只保留作证据，不是当前验证入口；禁止直接重放其中 GPU/服务命令。
- 导入镜像、离线构建、隔离恢复和 FPGA 精度分别验收。更新本地文档清单不提高恢复状态。
- CURRENT_STATE、GUIDE 和 manifest 只作恢复摘要及材料路线；动态事实按[来源表](../docs/platform/SOURCES.md)维护，不复制完整缺口或版本表。
- 修改文件后发布版本化校验清单，并在 README 指明当前版本；原 SHA256SUMS 保持不变，不能用新哈希覆盖历史验收。维护按[文档任务卡](../docs/agent/tasks/DOCUMENTATION.md)执行。不得写入凭据。
