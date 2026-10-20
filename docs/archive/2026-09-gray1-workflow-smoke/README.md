# Gray1 端到端工作流烟测归档

> 归档日期：2026-09-18  
> 原始位置：`docs/workflow-test/`  
> 执行时间：2026-09-12 至 2026-09-14  
> 状态：历史 smoke 证据，不是当前运行手册或精度基线

## 1. 这次烟测做了什么

该工作流使用确定性生成的 84 张、21 类 R-only 合成图片，验证以下软件链路：

```text
合成 R-only PNG 和 YOLO Pose 标签
-> grayprep 生成 net1280 单通道 Gray1 数据及 cam2000 BIN
-> 单张 H200 GPU 运行 1 epoch 训练
-> 生成 checkpoint 并完成 final evaluation
-> 准备量化任务输入
```

它的目的只是证明预处理、训练入口、checkpoint 和评估工作流可以连通。合成图不代表真实业务分布，结果不能用于模型精度、量化损失、FPGA/HIL 或生产验收。

## 2. 归档文件

| 文件 | 含义 |
|---|---|
| `WORKFLOW_README.md` | 原目录入口说明，保持历史内容 |
| `generate_synthetic_pose21.py` | 84 张确定性合成数据生成器 |
| `DATASET_INVENTORY.json` | 数据、工具、镜像和校验身份 |
| `DATASET_VERIFIED.txt` | 当时发布门禁结果 |
| `EXECUTION_RECORD.md` | 数据准备过程、失败修复和发布证据 |
| `run-gray1-training-smoke.sh` | 单卡 1 epoch 训练脚本 |
| `RUNBOOK.md` | 当时的数据、训练和量化命令 |
| `END_TO_END_QUICKSTART.md` | 当时的端到端操作说明 |

## 3. 已确认的远端证据

2026-09-18 只读复核时，以下内容仍存在于 H200：

```text
合成数据：/data1/ywang/workflow-test/synthetic-pose21-r-only-v1
成功运行：/data1/ywang/workflow-test/runs/e2e_smoke_20260914_164609
训练脚本：/data3/ywang/yolo-gray1-train/run-gray1-training-smoke.sh
旧预处理镜像：ywang/yolo-gray1-data-pipeline:0.1.1
```

历史成功运行使用 `imgsz=1280`、`batch=4`、`workers=4`、单卡和 1 epoch。它只证明 workflow smoke 成功，不代表正式训练完成条件。

## 4. 为什么不再作为当前文档

- 数据是合成 R-only 输入，不是当前正式 `pose21-gray1-1280-m-v1` 数据集；
- 文档固定了 grayprep `0.1.1` 和旧 repository commit，当前代码版本已是 `0.2.1`；
- 量化命令和 `generate_bin` 状态来自当时的平台现场；
- 文档中的 GPU、`/data1`、API 和容器命令必须重新经过当前授权及实时门禁；
- 当前正式 Gray1 训练和 6,600 张独立 test 已有单独记录，不能由本 smoke 替代。

## 5. 当前使用边界

不得直接复制执行本归档中的命令。若需要重新做 workflow smoke，应从当前仓库、镜像、数据合同和运行门禁重新生成命令，并使用新的任务目录。

当前依据：

- `../../../AGENTS.md`
- `../../agent/CONTEXT.md`
- `../../agent/RUNBOOK.md`
- `../../class-21/experiments/20260917_FORMAL_GRAY1_TRAINING.md`
- `../../class-21/experiments/20260918_19_EVALUATION_AND_QUANTIZATION.md`
