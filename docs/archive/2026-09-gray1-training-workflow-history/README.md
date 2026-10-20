# Gray1 训练工作流历史

原资料来自 docs/train，描述 2026-09-08/09 的环境调查、迁移计划和小规模实验。本目录只作历史溯源。

- [迁移计划摘要](agent-plan-chenshiwen-yolo-env-and-remote-data-workflow.md)：为什么隔离 owner 项目、如何规划数据与环境。
- [Step 0–16 摘要](agent-runbook-ywang-yolo-gray1-training-workflow.md)：执行结果、失败边界与后续替代。
- chenshiwen-yolov8-pose-21-landmarks.md 与 human-runbook-h200-nas-automount.md：保留原调查和当时挂载说明，不直接执行。
- yolo_full_image_inventory.csv：66,006 条源图记录，无文件哈希，不能替代数据备份校验。

2026-10-18 压缩两份长叙事，原文和原 SHA256SUMS 在[原文包](../2026-10-documentation-consolidation/original-documents.zip)中按原路径保存。当前 [SHA256SUMS](SHA256SUMS) 校验摘要及未改写资产；它不再表示两份摘要与原文逐字节相同。当前系统从[项目入口](../../class-21/README.md)读取。
