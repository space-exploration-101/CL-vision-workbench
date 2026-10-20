# Gray1 量化环境试验归档

> 归档日期：2026-09-18  
> 原始位置：`h200-worktree/rebuild/gray1-quant/`  
> 状态：历史证据，不是当前构建目录或执行入口

## 1. 归档原因

该目录是 2026-09-09 为旧 Gray1 训练实验准备单通道量化环境时形成的未跟踪工作区。它长期放在 H200 平台本地镜像中，容易被误认为当前平台源码或可直接执行的 rebuild 模块。

检查时确认：

- 本地镜像 HEAD 为 `5632157`，属于 2026-09-04 的旧平台版本；
- H200 当前平台 HEAD 为 `c45eb80`；
- H200 上原构建目录 `/data3/ywang/yolo-gray1-quant-env` 已不存在；
- H200 上历史镜像 `ywang/yolo-gray1-quant:ultralytics-8.3.98-v1` 已不存在；
- H200 当前平台仓库没有 `rebuild/gray1-quant` 或其中的任务准备、验证脚本；
- 原目录包含 87 个文件、543,658 字节，是现存唯一完整源码和失败重试现场。

因此原目录从 `h200-worktree` 移出并完整保存到 `raw/`。没有把它合入当前代码，也没有把历史路径改写成当前路径。

## 2. 它曾解决什么问题

这套试验环境从训练镜像 `ywang/yolo-gray1-train:ultralytics-8.3.98-v5` 派生，用于验证真正单通道 Gray1 量化运行时：

```text
输入：[1,1,1280,1280]
输出：[1,31,34000]
类别：21
关键点：[2,3]
```

对应历史实验：

```text
/data3/ywang/yolo-pose-experiments/20260909_192011_gray1_21c_1680_e50
历史 best.pt SHA-256：292cb13d75cdd571053b92e07f91b03e9e80c0b76b691b022065f445ff1f527f
```

Step 16 完成了隔离量化环境、PT 指标复现、双 ONNX 导出和 Gray1 ABI 对齐。Step 14 随后尝试接通标定、量化、FPGA 输入回环和 BIN 流程，产生多轮重试脚本和平台源码快照。

## 3. `raw/` 内容

值得保留的证据：

- `Dockerfile`：Step 16 使用的初始 Gray1 量化镜像构建定义；
- `verify_gray1_quant_runtime.py`：固定运行时版本和本地 ONNX Runtime 补丁检查；
- `verify_gray1_pipeline.py`：单通道预处理、PNG/BIN 回环、任务 ABI 检查；
- `verify_step16_ab.py`：训练镜像与量化镜像的 PT/ONNX A/B 验证；
- `finalize_step16.py`：Step 16 完成记录生成器；
- `prepare_step14_job.py`：旧实验的 400 张标定图、240 张测试图任务准备逻辑；
- `src/quantitize-platform/`：当时用于构建镜像的平台 pipeline 和本地补丁快照。

仅作失败过程参考的内容：

- `Dockerfile.v2` 至 `Dockerfile.v6`；
- `start_step14_retry2.sh` 至 `start_step14_retry5.sh`；
- `inspect_onnx_outputs.py`。

其中 `Dockerfile.v2` 至 `Dockerfile.v6` 引用原目录中不存在的 `src-v2`，不能作为完整、可复现的构建入口。重试脚本还写死了旧实验路径、旧镜像版本、用户 UID 和历史 GPU UUID。

## 4. 当前使用边界

不得直接使用 `raw/` 量化当前 21 类正式 Gray1 权重，原因包括：

1. 它对应旧实验和旧权重哈希，不是当前 `622cff54...` 权重；
2. 内嵌平台源码早于 H200 当前平台版本；
3. 任务数据选择是旧的 400/240 方案，不是当前 462 张标定和完整 6,600 张回归方案；
4. 多个 Dockerfile 和重试脚本不完整或写死历史环境；
5. 它没有证明当前权重的 FP16、QDQ、FPGA BIN 或实机结果。

当前执行依据：

- `../../class-21/knowledge/03_训练评估量化与FPGA流程.md`
- `../../class-21/experiments/20260918_19_EVALUATION_AND_QUANTIZATION.md`
- `../../agent/CONTEXT.md`
- `../../agent/RUNBOOK.md`

如需复用历史实现，应从 `raw/` 提取最小逻辑，针对 H200 当前 HEAD 重新实现和测试，不应恢复整个源码快照覆盖当前仓库。

## 5. 相关记录

Step 16 的完成结果也记录在：

[`../2026-09-gray1-training-workflow-history/agent-runbook-ywang-yolo-gray1-training-workflow.md`](../2026-09-gray1-training-workflow-history/agent-runbook-ywang-yolo-gray1-training-workflow.md)

该 runbook 保存结果和产物身份；本归档保存生成这些结果的原始实现与重试现场。
