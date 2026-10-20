# Agent 任务索引

先读 [CONTEXT](CONTEXT.md) 与根 [AGENTS](../../AGENTS.md)。本页不要求加载全部任务卡。

任务卡维护步骤与门禁，任务描述使用[统一模板入口](../AGENT_GUIDE.md#统一模板入口)；工具选型查[身份总索引](../../tools/README.md)，脚本存在不表示允许执行。

人类按任务选择路线可读[日常平台运维](../operations/README.md)；Agent 直接选择下表单张卡。动态事实查[来源表](../platform/SOURCES.md)，不从操作示例或历史报告推断实时状态。

| 任务 | 必须按需读取 | 完成标准 |
|---|---|---|
| 文档整合、链接与归档 | [DOCUMENTATION](tasks/DOCUMENTATION.md) | 来源、链接、时效、归档校验一致 |
| 本机资料 Git 管理 | [版本管理](../VERSION_CONTROL.md) | 白名单、LFS、敏感信息与暂存清单检查；提交/远程/推送须明确授权 |
| 服务、GPU、任务只读排障 | [DIAGNOSTICS](tasks/DIAGNOSTICS.md) | 状态、进程、日志和资源交叉验证 |
| 启动或重启服务 | [SERVICE](tasks/SERVICE.md) | 授权、任务保护、health/ready 与历史查询 |
| 评估、量化、FPGA 比较 | [EVALUATION](tasks/EVALUATION.md) | 同模型同数据，分别验收软件与硬件 |
| NAS 备份、归档和清理 | [BACKUP](tasks/BACKUP.md) | .partial、哈希、独立回读、恢复依据 |
| 灾难恢复 | [恢复手册](../reference/RECOVERY.md) + [BACKUP](tasks/BACKUP.md) | 固定版本的隔离恢复 |
| 环境重建 | [重建指南](../../rebuild/REBUILD_GUIDE.md) | 构建链与运行验收分别证明 |
| 性能优化 | [性能指南](../reference/PERFORMANCE.md) | 固定基线 A/B 与精度回归 |

仅本地文档任务不需要连接服务器。已有授权范围内继续执行；扩展到 GPU、重启、NAS 写入或删除前确认授权覆盖。结束报告实际变更、验证、未完成项和下一步；未现场复核的状态不得写成今日已验收。
