# 文档维护规则

继承根目录 [AGENTS.md](../AGENTS.md)。本文件只规定文档，不授予服务器、GPU 或 NAS 操作权限。

- 先选读 [DOCUMENTATION](agent/tasks/DOCUMENTATION.md)，再读本次涉及目录的 README。
- README 面向人类：先交代目录用途、必要术语和阅读路径；AGENTS 面向自动化执行：保持简短，链接到独立任务卡。
- 当前事实写最近证据日期、适用模型/数据和未验收项。修改摘要时同步关联入口，不改写原始实验数值。
- 动态事实按[来源表](platform/SOURCES.md)只维护一处详细说明；冲突按[文档任务卡](agent/tasks/DOCUMENTATION.md)登记，不合并猜测值。reference、operations、rebuild 分别承担通用方法、日常运维和完整恢复材料路线。
- 被替代的设计和运行过程移入归档；仍用于当前模型的证据留在 reports/current 或 operations/runs，并明确证据时点。
- 原始报告附件、哈希清单、模型和数据不因文档整理被重写。改动受校验保护的文档时保留原清单和副本，再更新对应清单。
- 21类当前报告在 `class-21/reports/current/`；地球、月球两类材料独立放在 [Earth_Moon](Earth_Moon/README.md)。21类正式附件见 [outputs/class-21](../outputs/class-21/README.md)，过渡实验见 [历史归档](archive/2026-09-class21-transition/README.md)。
- 不遍历 work；默认不展开 archive/raw、outputs/class-21 和其他大附件。历史授权不是当前任务授权。
