# 端到端工作流烟测

本目录记录H200上从数据准备、gray1预处理、训练到量化的可复现烟测流程。

## 数据边界

- 不读取、复制、链接或派生任何路径名或所有权与`chenshiwen`有关的数据。
- 当前数据由`generate_synthetic_pose21.py`确定性生成，仅用于流程测试，不用于精度结论。
- 远端根目录：`/data1/ywang/workflow-test/synthetic-pose21-r-only-v1`。
- 当前兼容目标：21类YOLO pose、`kpt_shape=[2,3]`、R-only源PNG，经grayprep输出真正单通道gray1。

## 文档

- `generate_synthetic_pose21.py`：确定性源数据生成器。
- `RUNBOOK.md`：数据、训练和量化命令。
- `EXECUTION_RECORD.md`：本次数据准备的实际证据和hash。
- `DATASET_INVENTORY.json`：远端数据的机器可读清单。
- `DATASET_VERIFIED.txt`：发布门禁标记的来源文件。

## 完成边界

本次只准备和验证数据，不启动GPU、训练或量化。后续训练只验证工作流；量化平台当前`/ready`异常且`generate_bin`存在已知失败，因此不能宣称生产或FPGA验证完成。现有shared datasets可能具有其他来源，本工作流明确禁止使用它们；量化输入必须从本合成数据集打包。
