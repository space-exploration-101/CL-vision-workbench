# 备份运行记录

面向查证备份覆盖的工程师与 Agent；按资产类型选择报告，不必读取全部历史日志。镜像、数据、模型和结果各有独立覆盖时间点。

需要按时间线了解背景、解包使用数据或查用脚本时，从 [rebuild/data-backup 统一资料入口](../../../../../rebuild/data-backup/README.md) 开始。该入口收录带来源哈希的本地脚本/说明快照；本目录原始报告保持原位。

本目录保存已经实际执行的备份与校验报告，不保存大文件资产。NAS 上的 manifest、
`SHA256SUMS` 和状态标记是归档身份的权威来源。当前设计与恢复边界以
[NAS 备份与环境恢复](../../../../platform/01_NAS备份与环境恢复.md)为准；执行步骤从[备份任务卡](../../../../agent/tasks/BACKUP.md)开始，不直接照抄原报告命令。历史前因后果见[归档摘要](../../../../archive/2026-10-operations-history/README.md)。

| 原始证据 | 仍有效的用途 | 适用边界 |
|---|---|---|
| [auto_landmark Dataset 归档](raw-dataset/auto-landmark-Dataset-20260922-161828/completion-report.md) | 09.22–23 快照发布、抽样恢复及独立校验 | 快照 ID 固定；不等于所有后续模型数据已归档，nas-docs 按包内原版本解释 |
| [旧 R-only 归档与清理](historical-r-only/20260925-local-data-cleanup/completion-report.md) | 09.25 旧权重与选择清单、删除前恢复依据 | 模型已被 Gray1 替代，归档凭证仍保留；不能重放删除动作 |
| [10.16 四类镜像归档](docker-images/20261016/completion-report.md) | 镜像/源码身份及 ASSET_VERIFIED 时点 | 原报告未证明整套恢复；其中旧 knowledge/04 链接按[迁移映射](../../../../archive/2026-10-platform-docs/README.md)解释 |

原报告、JSON、nas-docs 复制件与清单不改写；不同资产按各自时点维护证据。新增源码和后加输出是否被覆盖，只查平台 01，不在此另立当前状态表。混合历史状态的适用/替代边界见归档摘要，不因报告完成就移除恢复凭证。
