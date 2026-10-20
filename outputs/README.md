# 本地实验附件与生成材料

这里集中保存需要按具体任务查阅的文件。**正式实验附件**是报告引用的模型、结果和测试证据；**分析过程/来源材料**保留分析输入、计算实现、来源版本及过程记录；**展示预览**用于阅读图表；**依赖链接**仅定位运行依赖，不是实验凭证。分类按文件用途，不按目录名一刀切；四类都不自动授予删除或执行权限。

| 位置 | 用途与边界 |
|---|---|
| [class-21/](class-21/README.md) | 当前 21 类项目的正式实验附件；内容的结论与限制以[报告](../docs/class-21/reports/current/README.md)为准 |
| 现有 `fpga_*` 目录 | 分析来源、脚本、工作簿、统计记录与展示图片混存；按下表区分，全部保持原位 |

已被替代的实验保存在 [docs/archive](../docs/archive/2026-09-class21-transition/README.md)，不混入当前附件。新任务按专题/任务分目录；需要复查时指定具体文件，不批量展开 `node_modules` 或其他外部依赖目标。本次未改变现有预览及其依赖链接。

本机 Git 仅跟踪文档、数据与日志；下表脚本、依赖及明确展示预览不进入版本库，但仍原位保留，不因此变为可删除材料。历史对照图与来源工作簿继续保留；模型、BIN、证据 ZIP 使用 LFS。克隆不包含全部生成实现，不能据此承诺重现分析，详见[版本管理](../docs/VERSION_CONTROL.md)。

## 原位材料分类索引

下表按 2026-10-19 的文件名、类型和外层引用编制；未解析工作簿/统计内容、未运行脚本、未作图像视觉验收。工作簿中的 original、before 或 input 仅说明其来源/过程角色，不足以证明其中内容与正式附件相同。

| 原位置 | 分析过程/来源材料（保护原文） | 展示类文件（不承诺可再生成） | 依赖链接 |
|---|---|---|---|
| `fpga_content1000/` | original.xlsx、before_publish.xlsx、input.xlsx、audit.json、build.mjs、prepare.py、verify.py | Ground_Truth.png、Onnx.png、Precision_Loss.png | node_modules，未进入 |
| `fpga_full500/` | build.mjs、csv_update.json、verification.json | FPGA.png、Ground_Truth.png、Onnx.png、PT.png | node_modules，未进入 |
| `fpga_gt_anchored/` | original_report.md、section.md、statistics.json、classification_counts.json、add_classification.py、analyze.py、build_report.py、verify.py | 无独立图片文件 | 无 |
| `fpga_pixel_statistics/` | report_original.md、section.md、statistics.json、tail_statistics.json、analyze.py、build_report.py、update_tail_report.py | 无独立图片文件 | 无 |
| `fpga_precision_loss/` | original.xlsx、before_publish.xlsx、audit.json、build.mjs、verify.py | Precision_Loss.png、Precision_Loss_metrics.png；before_FPGA.png 兼有历史对照用途 | node_modules，未进入 |
| `fpga_precision_wide/` | original.xlsx、before_publish.xlsx、input_values.xlsx、expanded_formulas.json、audit.json、build.mjs、prepare.py、verify.py | GT_FPGA.png、GT_ONNX.png、GT_PT.png、ONNX_FPGA.png、PT_ONNX.png；before.png 兼有历史对照用途 | node_modules，未进入 |
| `fpga_preview_001/` | build.mjs 是生成实现，不是图片或删除授权 | FPGA.png、Ground_Truth.png、Onnx.png、PT.png | node_modules，未进入 |
| `fpga_stage_statistics/` | original_report.md、section.md、statistics.json、analyze.py、build_report.py | 无独立图片文件 | 无 |

正式目录中的图像、CSV/XLSX、JSON、脚本也可能是报告证据的一部分，不能按扩展名套用本表后删除。[class-21 索引](class-21/README.md)负责精确报告关系；正式 `FPGA_logs/summary_with_mapping.xlsx` 与本表过程工作簿不互相替换。历史来源报告不能覆盖 reports/current 中保留的 10.19 统计版本。

## 预览不等于可删除材料

`fpga_preview_001` 等目录中的展示图片属于预览用途；是否能够重新生成，还需对应源数据、脚本和依赖齐备并实际验证。本轮没有执行生成器，不能承诺这些文件均可再生成。

`fpga_stage_statistics`、`fpga_pixel_statistics`、`fpga_gt_anchored` 等目录还含 original_report.md、report_original.md、section.md、分析脚本或统计 JSON。这些是分析过程及来源材料，不能与纯展示图片一起按“可再生成”删除或压缩。它们保留原位置、内容和依赖链接；当前模型结论仍从正式报告及其精确附件读取。

## 本轮核验边界

第 6 阶段采用“原位分类并验证”，不重复迁移附件，不搬动 operations/runs、rebuild/legacy、task-artifacts。按两次迁移清单对正式附件流式校验 SHA-256；只验证文件身份，不解包模型、不重算实验结果。分析/展示目录只列一级文件与链接属性，未进入依赖链接。范围、精确映射、原文副本与回滚见[第 6 阶段记录](../docs/archive/2026-10-attachment-closeout/README.md)。
