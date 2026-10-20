# 2026-10-18 本地文档整合

本次仅整理本机文档与导航，没有访问 work、连接 H200/NAS、使用 GPU 或操作远端服务。

## 整理前副本

完整副本：`E:\codex\quantitize-backups\20261018-011353-before-docs-consolidation`，原始目录内容在其中的 `content/`。

4,539 个普通文件，共 6,294,069,314 bytes，全部与源文件 SHA-256 一致；248 个目录和 5 个依赖目录链接的路径/目标一致。排除 work，链接保留为链接，外部目标内容不展开。副本包含 SHA256-manifest.csv、links.json、backup-summary.json 与 VERIFIED。它是本机回滚点，不是异机灾备。

## 实际整理

- 保留 docs/class-21、reference、agent、archive 和 rebuild 主干；区分人类入口、Agent 任务卡、现用证据与历史材料。
- 将 CONTEXT/RUNBOOK 拆为短索引及 topics/tasks，避免每次加载操作流水账。
- 更新 main 与运行目录的区别、10.16 备份覆盖、API readiness 待复核、weight-only 基线；保留 FPGA/全量回归/隔离恢复的未完成状态。
- 为人类入口先解释模型、训练、量化、镜像、数据与验收术语。
- 压缩 9 月时间线和两份训练叙事；10 月过程集中到 [OCTOBER_CONTEXT](../OCTOBER_CONTEXT.md)，过期命令不再处于当前恢复入口。
- 实验模型、数据、tensor/BIN、CSV 和已有工具脚本保持内容；临时与任务快照只补外层定位，不从中推断当前状态。

## 移动映射

| 原位置 | 归档位置 |
|---|---|
| docs/reference/PERFORMANCE_OPTIMIZATION_FINAL_20260903.md | [9 月性能原报告](../2026-09-performance/PERFORMANCE_OPTIMIZATION_FINAL_20260903.md) |
| docs/reference/backup/BACKUP_STATUS.md | [旧系统备份状态](../2026-09-migration/backup/BACKUP_STATUS.md) |
| rebuild/manifests 与 rebuild/snapshots | [pre-gray1-20260902](../../../rebuild/legacy/pre-gray1-20260902/README.md) 下的同名目录 |
| rebuild/verify_host.sh 与 verify_runtime.py | [gray1-20260918](../../../rebuild/legacy/gray1-20260918/README.md) |

原路径到新路径的文件身份对比以本目录校验报告为准。归档的脚本和快照不作为当前可执行流程。

## 原文与回滚

[original-documents.zip](original-documents.zip) 按整理前相对路径保存本次改写的主要文档和完整原 rebuild 目录。[original-documents.json](original-documents.json) 记录每个成员的长度、原 SHA-256 与 ZIP 哈希；压缩摘要不能替代原始验收证据。

需要还原个别文档时，从 ZIP 或完整副本提取到新临时目录，核对哈希并比较，再恢复明确文件。整目录回滚应先保存整理后版本，不使用镜像删除命令，以免影响 work 或用户后来新增的内容。

## 验证与后续

本目录保存 [链接校验](validation.json)、[变更和文件身份核对](change-audit.json) 与 [SHA256SUMS](SHA256SUMS)。链接检查覆盖活跃 Markdown 和历史入口，不对原始快照内部旧链接或外网 URL 宣称有效。

远端状态未重新检查；后续运行、恢复与部署按当前任务卡进行。本文的文档整理结果不提高业务或灾备验收状态。
