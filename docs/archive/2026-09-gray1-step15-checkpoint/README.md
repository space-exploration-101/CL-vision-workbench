# 2026-09-09 Gray1 Step 15 历史 checkpoint

> 归档日期：2026-09-18  
> 状态：历史证据，不是当前训练、评估或量化输入

## 内容

`best.pt` 来自早期小规模 Gray1 集成实验：

```text
实验：20260909_192011_gray1_21c_1680_e50
阶段：step15_gray1_21c_1680_e50
字节数：199,108,234
SHA-256：292cb13d75cdd571053b92e07f91b03e9e80c0b76b691b022065f445ff1f527f
```

原记录路径：

```text
/data3/ywang/yolo-pose-experiments/20260909_192011_gray1_21c_1680_e50/runs/step15_gray1_21c_1680_e50/weights/best.pt
```

2026-09-18 只读复核时，该 H200 路径及同尺寸 `best.pt` 已不存在。因此没有删除本地副本，而是从容易与当前资料混淆的顶层 `class-21/` 移入本归档。

## 使用边界

- 当前正式 50 epoch Gray1 权重的 SHA-256 是 `622cff54db472b61c85db03648b36c119f030eacbd4cd87cf5d2dda67e69d6a7`，不得与本文件混用。
- 本 checkpoint 不得作为当前量化、FPGA 打包或精度验收输入。
- 历史训练与 Step 16 量化环境记录见：
  - `../2026-09-gray1-training-workflow-history/agent-runbook-ywang-yolo-gray1-training-workflow.md`
  - `../2026-09-gray1-quant-env/README.md`
