# Gray1 Predictor 修复与独立 test 脚本归档

> 归档日期：2026-09-18  
> 原始位置：`.codex_tmp/`  
> 状态：历史实现与试验现场，不是当前直接执行入口

## 1. 归档范围

本目录保存 2026-09-18 为 21 类单通道 Gray1 模型准备独立 test、定位 Predictor 三通道假设并构建修复镜像时产生的 14 个脚本和配置文件。

这些文件从临时目录迁出后按用途分为：

```text
raw/
├── test-eval/          独立 test 评估器、GPU 等待和运行 wrapper
├── predictor-fix/      Predictor 通道/预处理修复、Dockerfile 和 CPU 冒烟
└── legacy-epoch75/     被合同门禁阻断的旧 R-only 权重评估现场
```

原始文件保持不变。归档只增加本说明，不把历史路径或镜像名称改写成当前值。

## 2. Predictor 修复

原 Ultralytics Predictor 有三处不适用于真正单通道模型的行为：

1. warmup 输入写死为 3 通道；
2. OpenCV LetterBox 可能把 `HxWx1` 压成 `HxW`；
3. preprocess 无条件执行 BGR 到 RGB 的通道翻转。

`raw/predictor-fix/` 保存修补脚本、两阶段 Dockerfile 和 CPU 冒烟脚本。最终修复逻辑是：

- warmup 通道优先读取模型 metadata，回退到模型 YAML 的 `ch`；
- LetterBox 后为单通道结果恢复末尾通道维；
- 仅当末维为 3 时翻转 BGR/RGB，单通道只转换为 BCHW。

最终冒烟证据：

```text
FIRST_CONV_IN_CHANNELS=1
WARMUP_IMGSZ=(1,1,1280,1280)
PREPROCESS_SHAPE=(1,1,1280,1280)
SMOKE_GRAY1_PREDICT_PASS
```

后续完整 test 使用的修复镜像是：

```text
ywang/yolo-gray1-train:ultralytics-8.3.98-v1-predictor-gray1
```

## 3. 独立 test 脚本

`raw/test-eval/eval_pixel_errors.py` 是核心评估器，支持：

- 验证冻结权重身份和 Gray1 数据合同；
- 保存逐图预测和 YOLO 格式预测标签；
- 按同类、IoU 不低于 0.5、置信度降序贪心一对一匹配；
- 统计 bbox 中心及两个关键点的像素误差；
- 输出匹配、漏检、误检、逐图和总体 CSV/JSON。

该目录中的 `freeze_and_setup.sh` 和 `run_test_eval.sh` 属于首次失败运行：

```text
pose21_gray1_m_50e_test_eval_20260918_203306
```

其中 `run_test_eval.sh` 仍写死未修复的原始训练镜像 `...-v1`，只能用于解释失败原因，不得作为最终运行入口。最终成功运行使用了新的运行目录和 `...-v1-predictor-gray1` 镜像：

```text
pose21_gray1_m_50e_test_eval_20260918_212142
```

成功结果以 `docs/class-21/reports/current/20260920_Pose21_Gray1_训练测试标定与量化完整报告.md` 为准，而不是根据本归档中的 wrapper 推断。

## 4. 旧 epoch75 现场

`raw/legacy-epoch75/` 保存一次被输入合同门禁正确阻断的旧权重检查：

```text
运行：epoch75_strip_bbox_center_eval_20260918_152813
状态：BLOCKED
```

阻断原因包括：

- checkpoint 为 22 类，而目标数据集为 21 类；
- checkpoint 类别表含 `liuniwang`，后续类别 ID 与当前表错位；
- checkpoint 为 `kpt_shape=[1,3]`，当前数据集为 `[2,3]`；
- 输入语义是历史三通道 R-only，不是当前单通道 Gray1。

这些文件只用于证明门禁和保留问题定位过程，不得用于当前模型评估。

## 5. 当前使用边界

本归档中的修补方式依赖当时的 Ultralytics 8.3.98 源码形态，且 shell wrapper 写有历史 H200 路径、镜像和运行 ID。复用前必须针对当前代码重新审阅和测试，不能直接执行。

当前事实与执行依据：

- `../../class-21/README.md`
- `../../class-21/experiments/20260918_19_EVALUATION_AND_QUANTIZATION.md`
- `../../class-21/knowledge/03_训练评估量化与FPGA流程.md`
- `../../agent/CONTEXT.md`
- `../../agent/RUNBOOK.md`

当前正式权重：

```text
/data3/ywang/yolo-gray1-train/runs/pose21_gray1_m_50e_20260917_113127/experiments/pose21_gray1_m_50e_20260917_113127/weights/best.pt
SHA-256：622cff54db472b61c85db03648b36c119f030eacbd4cd87cf5d2dda67e69d6a7
```
