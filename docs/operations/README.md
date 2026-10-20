# 日常平台运维入口

运维维护现有服务、任务与存储的可用性。**诊断**先收集证据，**变更**可能影响任务，**验收**确认预期功能并记录剩余问题。health 表示服务能响应，ready 表示具备处理任务的条件，两者需要分别检查。

这里仅放平台共用的任务路线，不保存模型接口、实验指标或新的资产表。现有执行记录、归档工具和模板保留原位置，避免改变历史引用和任务包身份。

## 按任务选择最短路线

| 本次任务 | 先读事实 | 再读操作与验收 |
|---|---|---|
| 服务或任务异常，只读排查 | [HOSTS](../agent/topics/HOSTS.md)，需要版本时再读[源码身份](../agent/topics/REPOSITORIES.md) | [DIAGNOSTICS](../agent/tasks/DIAGNOSTICS.md) |
| 已授权启动或重启服务 | 上述诊断结果和受影响任务 | [SERVICE](../agent/tasks/SERVICE.md) |
| NAS 备份、归档或清理 | [备份短入口](../agent/topics/BACKUPS.md) → 所需资产章节 | [BACKUP](../agent/tasks/BACKUP.md) |
| 完整环境丢失、换主机或重新构建 | [通用恢复判断](../reference/RECOVERY.md) | 转到 [rebuild](../../rebuild/README.md)，不作为普通重启处理 |
| 模型评估或 FPGA 对比 | 转到 [class-21](../class-21/README.md)固定模型与数据 | [EVALUATION](../agent/tasks/EVALUATION.md)，与服务可用性分别验收 |

按需查[平台事实来源](../platform/SOURCES.md)，不要一次读取全部知识库。运行环境证据截至 10.16、源码证据截至 10.18；本入口没有现场验收结论。5090 是[TODO 计划](../platform/03_5090训练测试推理与量化运行环境.md)，不是现有运维目标主机已就绪的声明。

## 证据与工具

既有[运行记录](../class-21/operations/runs/README.md)按日期证明当时操作；[工具总索引](../../tools/README.md)逐项区分维护工具、可复用候选、冻结实验与已完成一次性脚本；[模板统一入口](../AGENT_GUIDE.md#统一模板入口)用于描述任务，步骤与门禁归任务卡，不构成授权。这里不复制工具或模板，不迁移原证据包，也不重写历史结果。

H200 使用 ssh H200 直连。执行前确认任务范围；GPU、重启、NAS 写入和删除须被授权覆盖。记录观察日期、实际动作、验证结果和未完成项；本机文档整理无需执行任何服务器命令。
