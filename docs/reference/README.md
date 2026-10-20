# 按需参考

这里解释通用概念、判断方法和验收层次，回答“怎样理解与判断”，不维护当前部署版本、资产覆盖或实验指标表。日常服务问题走[operations](../operations/README.md)，完整主机恢复和构建走[rebuild](../../rebuild/README.md)，动态事实查[来源表](../platform/SOURCES.md)。

- [基础教材](VISION_MODEL_QUANTIZATION_FPGA_PRIMER.md)：图像、张量、卷积、量化和 FPGA；从零建立理解。
- [恢复手册](RECOVERY.md)：服务或主机故障时怎样选择恢复路径。
- [性能指南](PERFORMANCE.md)：怎样固定条件比较速度、避免把旧实验当作当前指标。
- [备份入口](backup/README.md)：当前备份覆盖与旧系统快照的区别。
- [环境重建](../../rebuild/README.md)：镜像恢复和从源码构建的不同条件。

9 月 2 日恢复命令和 9 月 3 日性能结果已作为历史依据归档，不是当前默认步骤。最近恢复基线沿用 10.16 证据，具体覆盖只在[平台 01](../platform/01_NAS备份与环境恢复.md)维护；本次未执行恢复。
