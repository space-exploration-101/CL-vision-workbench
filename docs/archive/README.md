# 历史资料归档

本目录保存被替代的方案、历史实验与操作过程。优先阅读压缩后的因果摘要，只有需要具体日志、命令或身份时才展开原文。

## 阅读入口

- [10.20 工作区更名与 Git 初始化](2026-10-workbench-rename/README.md)：保留更名受阻时的原始审计；[后续完成记录](2026-10-workbench-rename/RENAME_COMPLETION.md)核对新根目录、Git 身份和已同步的 H200 控制端说明。旧路径及原证据不改写。

- [第 7 阶段全局验收与最终交接](2026-10-final-handoff/README.md)：整合完整基线与后续合法修改，说明最终身份、文档快照、校验边界及回滚；第 5 阶段已由 CLOSEOUT 收尾，不再等待移动表。

- [9 月摘要](SEPTEMBER_CONTEXT.md)：迁移、R-only → Gray1、训练、量化失败与恢复、数据保护。
- [10 月摘要](OCTOBER_CONTEXT.md)：工程测试、镜像统一、备份、main 合并和 5090 待办。
- [本次整理记录与原文包](2026-10-documentation-consolidation/README.md)：副本、压缩范围、移动映射和校验。
- [实验附件迁移记录](2026-10-artifact-relocation/README.md)：正式附件集中到 outputs，过渡实验移入 archive；包内内容与校验身份保留。
- [10.19 平台说明分离与附件补迁](2026-10-platform-docs/README.md)：保全 10.19 报告新基线，补充任务快照限制；旧审计清单不改写。
- [第 3 阶段当前文档整合](2026-10-current-docs/README.md)：动态事实分工、按任务阅读路线和冲突登记；原始证据不改写。
- [第 4 阶段工具与模板](2026-10-tools-templates/README.md)：逐项工具身份、统一模板入口、原脚本哈希与回滚记录；未执行业务脚本。
- [运维因果摘要与第 5 阶段记录](2026-10-operations-history/README.md)：区分有效凭证与被替代状态；用户已确认原位保留方案，[第 5 阶段已收尾](2026-10-operations-history/CLOSEOUT.md)，不再等待移动清单。

本目录不是当前操作手册。归档中的 `current`、`ready`、`latest`、旧路径和旧授权只代表写成时的状态。当前操作依次以根目录 `AGENTS.md`、`docs/agent/`、`docs/class-21/` 和实时检查为准。

## 主线摘要

| 日期 | 主线 | 结果 |
|---|---|---|
| 09-01 | 平台整理与源端备份 | Web/API 分层，迁移资产约 51.782 GiB 写入 NAS |
| 09-02 | H200 迁移与验收 | 八阶段 golden task 完成，生成 pre-Gray1 快照 |
| 09-03/04 | 21 类 R-only 过渡 | 定位 BGR2GRAY 故障，PT/ONNX/软件回环 F1 对齐 |
| 09-09 | Gray1 小规模链路 | 训练、PT/ONNX、隔离量化环境和历史 Step 15 checkpoint |
| 09-10/11 | 预处理性能 | 固定样本线程扫描，识别 PNG 解码和 Python 循环热点 |
| 09-12/14 | 标准化与 smoke | 84 张确定性合成数据、单卡 1 epoch 工作流贯通 |
| 09-16 | L0 软件工作包 | trace、outcome、test package 与集成合同完成 CPU 验证 |
| 09-17 | 正式 Gray1 数据与训练 | 26,794 张数据、1 epoch smoke、50 epoch 候选权重 |
| 09-18 | 独立 test 与归档 | Predictor Gray1 修复、6,600 张 test PASS、历史资产收拢 |
| 09-19 | Gray1 归档预检 | 因 `shared_data` 仍在写入而停止，NAS 写入为 0 |
| 09-20 | weight-only 恢复精度 | 462 张检测指标与 PT 一致，软件交付完成 |
| 10-16/18 | 镜像备份与源码合并 | 四类镜像已归档；main 已同步，原运行目录未部署 |

## 主题目录

| 目录 | 内容 | 使用边界 |
|---|---|---|
| [`2026-09-migration/`](2026-09-migration/) | 平台整理、源备份、H200 迁移、网络与恢复 | pre-Gray1 历史基线 |
| [`2026-09-class21-transition/`](2026-09-class21-transition/README.md) | 旧 2 类与 21 类 R-only → Gray1 过渡摘要、test5 原始材料 | 已被当前模型替代，仅供溯源 |
| [`2026-09-gray1-quant-env/`](2026-09-gray1-quant-env/) | Step 14/16 隔离量化源码与失败重试 | 旧平台/旧权重，不直接执行 |
| [`2026-09-gray1-predictor-test-eval/`](2026-09-gray1-predictor-test-eval/) | Predictor 修复和独立 test 脚本 | 当前结果以 `docs/class-21/` 为准 |
| [`2026-09-grayprep-performance-step5-14/`](2026-09-grayprep-performance-step5-14/) | OpenCV 线程扫描和 GPU PoC | 历史性能方法 |
| [`2026-09-gray1-workflow-smoke/`](2026-09-gray1-workflow-smoke/) | 84 张合成数据和 1 epoch smoke | 不代表精度或生产验收 |
| [`2026-09-standardization-work-packages/`](2026-09-standardization-work-packages/) | 工作板、设计和 WP08-A 快照 | L0 软件证据，不代表硬件通过 |
| [`2026-09-gray1-step15-checkpoint/`](2026-09-gray1-step15-checkpoint/) | 小规模历史 `best.pt` | 禁止替代正式 50e 权重 |
| [`2026-09-local-artifact-cleanup/`](2026-09-local-artifact-cleanup/) | 旧 2 类模型、文档、Gray1/FPGA 摘要 | 本地审计归档，不等于 NAS 备份 |
| [`2026-09-h200-worktree-retirement/`](2026-09-h200-worktree-retirement/) | 删除旧本机镜像前保留的未跟踪文件 | 来源/删除证据 |
| [`2026-09-sim-train-fpga-doc/`](2026-09-sim-train-fpga-doc/) | QP-TS-2026-001 技术说明及 OOXML 源 | 说明性文档 |
| [`2026-09-gray1-blocked-backup-run/`](2026-09-gray1-blocked-backup-run/) | Gray1 归档预检停止记录 | 审计证据，不是有效备份 |
| [`2026-09-gray1-training-workflow-history/`](2026-09-gray1-training-workflow-history/) | 旧训练环境调查、迁移计划、Step 0-16 记录和源图清单 | 历史证据，不是当前操作手册 |
| [`2026-04-cam2000-prototype/`](2026-04-cam2000-prototype/) | 早期 packed-u12 转换器和样本 | 协议背景，不是 golden |

新增主题：

- [9 月迁移入口](2026-09-migration/README.md)与[旧模型性能](2026-09-performance/README.md)。
- [原始 Gray1 迁移文档](2026-09-class21-migration-source-docs/README.md)。
- [10 月文档整理](2026-10-documentation-consolidation/README.md)。
- [数据归档、镜像、main 整合的因果链](2026-10-operations-history/README.md)：原证据仍在 operations/runs，旧 PR 步骤和标签状态不是当前操作指令。
- [rebuild/legacy](../../rebuild/legacy/README.md)保留历史重建/验证材料；[task-artifacts](../../task-artifacts/README.md)保留局部构建快照。二者不强制迁入 archive，适用边界见[关系摘要](2026-10-operations-history/README.md#历史重建材料与局部构建快照)。

## 最重要的边界

- `snapshot-20260902-182606` 是 pre-Gray1 快照。
- 2026-09-04 的 21 类量化成功属于三通道 R-only，不等于当前 Gray1 已量化。
- 历史 Step 15 权重 `292cb13d...` 不得替代正式权重 `622cff54...`。
- 84 张 synthetic smoke 只证明流程贯通。
- WP06/WP07/WP08-A 的 L0 PASS 不代表真实 GPU 量化、FPGA/HIL、硬件或生产发布通过。
- 2026-10-18 压缩了时间线和两份长篇训练叙事；原文及旧哈希保存在可校验 ZIP 与整理前完整副本中。模型、tensor、实验 CSV 和其他原始附件未重写。
