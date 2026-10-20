# 21 类 Gray1 项目

这里保存当前模型的知识、技术接口与实验依据。21 类指模型可识别的目标类别数；Gray1 指每个像素只有一个灰度值，而不是红绿蓝三个通道。模型同时输出目标框和两个位置点，因此称为 Pose 模型。

## 当前结论

依据 9 月 20 日至 10 月 18 日已有记录整理；本次未重新运行实验。

- 冻结模型使用 `pose21-gray1-1280-m-v1`，共 26,794 张图；输入 `[B,1,1280,1280]`（批量、通道、高、宽）。
- 已完成 6,600 张 PT test 和 462 张量化对比。最终 W8A16 只量化权重，激活保持 FP16；软件八阶段交付完成。
- 500 张工程测试已有 PT/ONNX 结果，存在训练场景重叠；逐层软件参考已生成。
- 真实 FPGA 精度与 6,600 张量化 ONNX 全量回归仍待验收。
- 四类镜像与对应源码已 NAS 归档；完整恢复仍为 `PARTIAL`。最新 main 与运行目录不同，详见[源码身份](../agent/topics/REPOSITORIES.md)。
- 未来数据更新地标点并更换来源，旧标签两个关键点重合的限制不能延续成新数据的验收结论。

## 按问题阅读

| 目标 | 入口 |
|---|---|
| 不懂算法，先理解概念 | [知识库](knowledge/README.md)中的 01–03 |
| 查模型层、输入输出、FPGA 数据布局 | [架构](architecture/README.md) |
| 查某个结果的样本、指标、限制 | [实验报告](reports/README.md) |
| 查模型、测试结果、逐层参考等原始附件 | [正式附件](../../outputs/class-21/README.md) |
| 查镜像、NAS、恢复与 5090 TODO 计划 | [平台说明](../platform/README.md)中的 01–06 |
| 日常平台运维 | [平台 operations](../operations/README.md) |
| 模型相关任务与既有执行证据入口 | [专题 operations](operations/README.md) |
| Agent 只需固定模型/数据身份 | [模型基线](../agent/topics/MODEL_BASELINE.md) |
| 查 R-only 到 Gray1 的原因 | [历史实验](../archive/2026-09-class21-transition/HISTORICAL_TRANSITION_EXPERIMENTS.md)、[历史时间线](../archive/SEPTEMBER_CONTEXT.md) |

报告保留原实验口径；机器合同中的待确认项不能当成验收结果。冲突时先按同一模型、数据和环境的证据日期核对，不把计划或旧文件名当作现场事实。原始迁移过程见[原文归档](../archive/2026-09-class21-migration-source-docs/README.md)。
