# Agent 最小上下文

2026-10-20 确认项目名 CL-vision-workbench 并建立根 Git 仓库；本机目录现为 `E:\codex\CL-vision-workbench`，核验见[完成记录](../archive/2026-10-workbench-rename/RENAME_COMPLETION.md)与 HOSTS。H200 控制端路径说明已同步并回读校验；以下业务状态仍沿用原证据日期。

## 最近记录的短摘要

- 模型证据 09.20、09.25、10.12–19：当前冻结模型为 Pose21 Gray1，软件交付和工程分析不等于完整 FPGA 验收。[模型基线](topics/MODEL_BASELINE.md)
- 环境与备份证据 10.16；本机 PT 临时 NAS 副本 10.20：四类环境资产已归档，恢复仍为 PARTIAL，服务就绪异常尚待后续验收。[运行环境](../platform/02_H200训练测试推理与量化运行环境.md)、[备份入口](topics/BACKUPS.md)
- 源码证据 10.18/10.20：最新 main 与 H200 原运行工作区分离；5090 已克隆并固定到 H200 运行提交，未恢复完整环境或部署新 main。[REPOSITORIES](topics/REPOSITORIES.md)
- 5090 的 `$HOME/CL01_02` 固定原图、PyTorch GPU 与适配训练/续训通过；10.21 eval462/cal462 正在执行，原 ORT 仅支持 CPU。用户取消 P4.4 的 6,600 张全量回归，当前 5/13、IN_PROGRESS；保留 AMP scale=1、显式 Gray1 输入和校准线程适配，服务与双向接管未验收。[计划](../platform/03_5090训练测试推理与量化运行环境.md)、[回归](../class-21/operations/runs/20261021-5090-eval462-quantization.md)、[主机](topics/HOSTS.md)

## 本次只读需要的一份

| 问题 | 入口 |
|---|---|
| 主机、服务地址和路径 | [HOSTS](topics/HOSTS.md) |
| 开发、运行、恢复源码区别 | [REPOSITORIES](topics/REPOSITORIES.md) |
| 固定模型身份与相关实验 | [MODEL_BASELINE](topics/MODEL_BASELINE.md) |
| 数据版本与预处理 | [知识库 02](../class-21/knowledge/02_21类目标数据与输入预处理.md) |
| 备份覆盖或恢复缺口 | [BACKUPS](topics/BACKUPS.md)，再选所需章节 |
| 执行步骤与验收 | [RUNBOOK](RUNBOOK.md)，只读对应任务卡 |
| 文档职责或事实冲突 | [来源表](../platform/SOURCES.md)、[待核对项](../platform/OPEN_QUESTIONS.md) |
| 本机路径、Git 跟踪范围与首次推送 | [HOSTS](topics/HOSTS.md)、[版本管理](../VERSION_CONTROL.md) |

本机是文档与证据目录，不是 H200 服务源码根仓库。21类当前报告原位保留；地球、月球两类材料见 [Earth_Moon](../Earth_Moon/README.md)。21类正式附件见 [outputs/class-21](../../outputs/class-21/README.md)；冻结包内旧路径按迁移映射解释。排除 work，不沿链接读取依赖，不加载全部历史或张量附件。共享主机与授权边界以根 AGENTS 为准。
