# 本地模型与临时产物清理归档

2026-09-18 清理 `E:\codex\quantitize\model.pt` 和 `E:\codex\quantitize\tmp` 前，从 5,124 个临时文件（约 1.48 GiB）中筛选了仍具有追溯价值的内容。共归档 19 个文件，约 230.36 MiB。

## 保留内容

- `model/`：旧 2 类 YOLOv8x-Pose-P6 R-only 基线模型，已重命名为 `yolov8x-pose-p6_2class-r-only_baseline_df30b9fb.pt`。
- `documents/`：2026-03-02 设计文档源稿、2026-09-11 需求规格修订稿，以及 2026-09-16 设计文档的 DOCX/PDF 最终候选稿。
- `evidence/gray1/`：Step 5 示意图、Step 8b 备份清单、Step 10 数据清单和 Step 12/13 摘要及指标。
- `evidence/fpga/`：FPGA 结构对比结果。
- `evidence/documents/`：设计文档状态对比和识别示例图。

## 未保留内容

清理时不保留可重建的 Python/文档渲染依赖、Chromium 临时配置、重复页面 PNG、多轮中间 DOCX/PDF、缓存、临时修补脚本和已被正式文档/远程仓库取代的流程脚手架。

`SHA256SUMS` 为删除源文件前逐项校验得到的完整 SHA-256 清单。该目录仅是本地归档，不代表 NAS 备份。
