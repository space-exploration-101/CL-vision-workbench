# 21 类灰度图独立 test 评估报告

> 评估日期：2026-09-18  
> 评估主机：H200 共享服务器  
> 评估对象：21 类 Gray1 Pose 模型的 6,600 张独立 test 集  
> 运行状态：PASS

## 1. 试验目的

本轮试验评估 50 epoch Gray1 Pose 模型在独立 test split 上的推理表现，重点统计检测框中心和两个关键点相对标签的像素误差。test 数据未参与训练曲线和权重选择，与 train、val 在 group 层面无重叠。

本报告记录 PT 原始权重结果，不包含 ONNX 导出、量化或 FPGA 性能结论。

相关文档：

- 训练：`docs/class-21/21类灰度图正式训练计划.md`
- 数据：`docs/class-21/21类灰度图训练数据预处理.md`
- 量化规划：`docs/class-21/21类Gray1模型量化与cam2000性能测试计划.md`

## 2. 模型与数据

### 2.1 模型

```text
训练运行：pose21_gray1_m_50e_20260917_113127
权重：/data3/ywang/yolo-gray1-train/runs/pose21_gray1_m_50e_20260917_113127/experiments/pose21_gray1_m_50e_20260917_113127/weights/best.pt
权限：只读
权重字节数：199,108,106
已冻结 SHA-256：622cff54db472b61c85db03648b36c119f030eacbd4cd87cf5d2dda67e69d6a7
冻结记录：同目录 FROZEN.yaml
输入：[B,1,1280,1280]
类别数：21
关键点：[2,3]
输入语义：gray1
```

该权重来自 50 epoch、从 Gray1 YAML 零初始化的短周期训练，训练输入为真正单通道图像。训练阶段未启用在线颜色或几何增强。评估只使用冻结的 `best.pt`，不使用哈希不同的 `last.pt`。训练结束时 val 指标仅作为参考：Box mAP50 0.962、Box mAP50-95 0.921、Pose mAP50 0.964、Pose mAP50-95 0.963。

### 2.2 独立 test 数据

```text
数据集：pose21-gray1-1280-m-v1
图像：6,600
标签：6,600
正样本图：5,533
空标签图：1,067
独立 group：360
图像格式：1280x1280、uint8、单通道 Gray1
```

test split 来自原始 test 6,600 张图，经 net1280 处理：先缩放到 1000x1000，再在四边各 padding 140 像素。train:test 和 val:test 的 group 重叠均为 0。

21 类顺序为：agaier、aipuha、aituosha、ansainusa、aoerhong、chapala、dibulong、didikaka、kanjialu、lupate、qinghai、rineiwa、sading、saersaer、saleikamei、taihu、taiwan、tana、tekesi、wennibeige、moon。

数据集根目录当前存在 `VERIFIED`，但不存在早期准备记录曾写到的 `ACCEPTED`。本轮以 `VERIFIED`、`dataset.yaml`、图像和标签数量及实际推理结果作为门禁；没有补建 `ACCEPTED`。

## 3. 推理环境与过程

### 3.1 Predictor 问题与修复

50e 训练和 val 使用 Validator，能够按 `dataset.yaml` 的 `channels: 1` 预热；独立评估使用 `model.predict()`，原训练镜像存在三个单通道问题：

1. Predictor 预热输入写死为 `[1,3,1280,1280]`，与第一层卷积 `[80,1,3,3]` 冲突。
2. OpenCV LetterBox 会把 `HxWx1` 挤成 `HW`。
3. 预处理默认执行 BGR 到 RGB 翻转，不适用于单通道输入。

修复文件：

```text
/data3/ywang/yolo-gray1-train/src/Yolo_fine/ultralytics/ultralytics/engine/predictor.py
记录时 SHA-256：0abae2ba9a3028f7204ff7af5a3822485c280fdd92e6a817ed3c8c41d55921a3
改动规模：19 插入 / 3 删除
```

修复后的行为为：预热通道优先读取 `metadata.channels`，否则读取模型 YAML 的 `ch`；LetterBox 后补回被挤掉的单通道维度；只有最后一维为 3 时才执行 BGR 到 RGB 翻转。

镜像演进：

| 镜像 | ID | 用途 |
|---|---|---|
| `ywang/yolo-gray1-train:ultralytics-8.3.98-v1` | `sha256:7a291a40a004767c3d44b781545e8faeb8afe4654f4c191658292829a0959d03` | 50e 原训练镜像，保持不动 |
| `ywang/yolo-gray1-train:ultralytics-8.3.98-v1-predictor-chfix` | `sha256:f2a45df2e7bad0a5742d0763563f983724b626f3edc53aeff22a2acc9b8024dd` | 只修预热的中间镜像 |
| `ywang/yolo-gray1-train:ultralytics-8.3.98-v1-predictor-gray1` | `sha256:edaa61047923e708db681416ed99d2d7b657a4c689d9c89f6b4dcdb549f6c68c` | 本轮完整评估镜像 |

`predictor-gray1` 从原 `v1` 增加一层，只替换上述 `predictor.py`。量化镜像 `quantitize-platform-api:latest` 原本已按模型 `ch` 预热，本次没有重建或修改量化 API。

### 3.2 单通道冒烟与失败现场

完整评估前使用 CPU 和一张 test 图验证修复镜像，结果如下：

```text
FIRST_CONV_IN_CHANNELS=1
WARMUP_IMGSZ=(1, 1, 1280, 1280)
PREPROCESS_SHAPE=(1, 1, 1280, 1280)
RESULT_ORIG_SHAPE=(1280, 1280)
RESULT_PRED_COUNT=1
SMOKE_GRAY1_PREDICT_PASS
```

旧运行目录 `pose21_gray1_m_50e_test_eval_20260918_203306` 保留了一次使用原 `v1` 镜像失败的现场，错误发生在三通道预热。该目录没有被复用或覆盖，不属于最终精度结果。

### 3.3 完整 test 运行

独立推理运行：

```text
运行 ID：pose21_gray1_m_50e_test_eval_20260918_212142
镜像：ywang/yolo-gray1-train:ultralytics-8.3.98-v1-predictor-gray1
镜像 ID：sha256:edaa61047923e708db681416ed99d2d7b657a4c689d9c89f6b4dcdb549f6c68c
GPU：GPU-a86513cc-4848-d073-b250-2268b9c3458c
开始推理：2026-09-18 21:22:47 CST
完成推理：2026-09-18 21:25:25 CST
状态：PASS
```

推理参数：

| 参数 | 值 |
|---|---:|
| imgsz | 1280 |
| batch | 8 |
| conf | 0.25 |
| NMS IoU | 0.7 |
| max_det | 300 |
| GT/预测匹配 IoU | 0.5 |
| 关键点置信度阈值 | 0.5 |

匹配采用同类、IoU 不低于 0.5、按预测置信度从高到低的贪心一对一匹配。只对成功匹配的实例计算 bbox 中心和关键点像素误差。推理不生成可视化图片，但保存每张图的预测记录和标签格式结果。

## 4. 总体结果

### 4.1 检测与匹配

| 指标 | 数量 | 比例 |
|---|---:|---:|
| GT 实例 | 5,533 | 100% |
| 预测实例 | 5,399 | - |
| 成功匹配 | 5,150 | GT 的 93.08% |
| 漏检 | 383 | GT 的 6.92% |
| 误检 | 249 | 预测的 4.61% |
| 匹配精确率代理值 | - | 95.39% |

这里的 93.08% 和 95.39% 是在固定 `conf=0.25`、同类 IoU 0.5 匹配规则下得到的召回与精确率代理值，不是 COCO mAP。

### 4.2 像素误差

| 指标 | bbox 中心 | 两个关键点合计 |
|---|---:|---:|
| 样本数 | 5,150 | 10,300 |
| mean | 4.725 px | 5.491 px |
| median | 1.575 px | 2.078 px |
| p90 | 8.886 px | 11.644 px |
| p95 | 20.159 px | 23.089 px |
| max | 187.452 px | 150.072 px |
| 1 px 内 | 33.44% | 21.24% |
| 2 px 内 | 58.68% | 48.50% |
| 5 px 内 | 82.74% | 77.89% |
| 10 px 内 | 90.95% | 88.44% |

两个关键点的整体表现接近：关键点 0 均值 5.491 px、P95 23.131 px；关键点 1 均值 5.492 px、P95 22.938 px。数据集中两个关键点标签彼此重合，因此二者数值接近符合当前标签语义。

## 5. 逐类结果

| 类别 | 匹配数 | 漏检 | 误检 | bbox均值(px) | bbox P95(px) | 关键点均值(px) | 关键点P95(px) |
|---|---:|---:|---:|---:|---:|---:|---:|
| agaier | 247 | 28 | 13 | 5.807 | 21.643 | 6.187 | 22.129 |
| aipuha | 244 | 26 | 13 | 11.243 | 51.391 | 12.196 | 47.880 |
| aituosha | 293 | 17 | 11 | 5.474 | 18.864 | 5.787 | 19.776 |
| ansainusa | 191 | 26 | 4 | 6.183 | 23.223 | 7.255 | 30.408 |
| aoerhong | 249 | 21 | 5 | 4.012 | 17.789 | 5.086 | 21.157 |
| chapala | 252 | 5 | 23 | 1.939 | 4.846 | 2.258 | 6.356 |
| dibulong | 213 | 26 | 5 | 4.496 | 12.318 | 4.502 | 12.102 |
| didikaka | 301 | 11 | 11 | 24.014 | 80.058 | 24.910 | 78.655 |
| kanjialu | 293 | 14 | 39 | 2.244 | 6.424 | 3.584 | 10.228 |
| lupate | 256 | 18 | 17 | 2.232 | 5.529 | 2.560 | 6.713 |
| moon | 196 | 4 | 0 | 0.541 | 1.155 | 0.978 | 2.124 |
| qinghai | 296 | 9 | 2 | 2.975 | 7.729 | 4.142 | 10.291 |
| rineiwa | 264 | 17 | 18 | 2.515 | 9.838 | 3.158 | 12.787 |
| sading | 242 | 23 | 22 | 3.537 | 9.080 | 4.308 | 12.108 |
| saersaer | 249 | 14 | 11 | 1.677 | 3.502 | 1.897 | 4.137 |
| saleikamei | 251 | 15 | 3 | 1.993 | 7.446 | 2.880 | 10.858 |
| taihu | 140 | 30 | 1 | 2.524 | 7.845 | 2.676 | 7.815 |
| taiwan | 238 | 38 | 9 | 2.976 | 8.800 | 4.343 | 13.183 |
| tana | 247 | 12 | 19 | 2.162 | 5.479 | 2.776 | 6.975 |
| tekesi | 271 | 11 | 12 | 4.042 | 21.809 | 5.886 | 25.712 |
| wennibeige | 217 | 18 | 11 | 1.617 | 5.612 | 2.387 | 8.873 |

## 6. 精度分析

1. 整体定位主体集中度较好。bbox 中心误差中位数为 1.575 px，82.74% 的匹配实例位于 5 px 内；关键点误差中位数为 2.078 px，77.89% 位于 5 px 内。
2. 长尾误差明显。bbox 均值 4.725 px 显著高于中位数，P95 达 20.159 px，最大值 187.452 px，说明少量大误差样本显著拉高均值。
3. `didikaka` 是首要问题类别，bbox 与关键点均值约 24 px，P95 接近 80 px；`aipuha` 次之，bbox 均值 11.243 px、P95 51.391 px。后续应优先抽查这两类的大误差样本，确认是否存在框尺度、padding 后目标位置、相似类别或标签质量问题。
4. `taiwan` 漏检最多，为 38 个；`taihu` 的匹配数最低且漏检 30 个，应结合该类 test 样本数量和目标外观检查召回问题。
5. `kanjialu` 误检最多，为 39 个；`chapala`、`sading` 也有较多误检。需要从 `false_positives.csv` 回查误检发生在正样本图还是 1,067 张背景图。
6. `moon` 最稳定，bbox 均值 0.541 px、关键点均值 0.978 px，且无误检；但它与地标类的视觉和标签分布不同，不应以该类表现代表全部类别。
7. val mAP 与本报告像素误差采用不同统计口径。val 指标用于训练过程模型选择，本轮结果用于固定阈值下的独立 test 定位误差分析，两者不能直接互相替代。

## 7. 产物说明

本轮运行结果包含：

```text
artifacts/predictions.jsonl              每张图的 GT、预测和匹配详情
artifacts/predictions/labels/*.txt       6,600 个预测标签文件
artifacts/matched_errors.csv             5,150 个匹配实例的像素误差
artifacts/per_image.csv                  6,600 张图的逐图统计
artifacts/missed_gt.csv                  383 个漏检实例
artifacts/false_positives.csv             249 个误检实例
artifacts/summary.json                   总体和逐类汇总
eval.log / wait.log / status.env         启动门禁、进度和最终状态
```

NAS 备份同时保存 test 图像、test labels、`dataset.yaml`、数据集验证标记、冻结 `best.pt`、冻结记录、完整评估运行目录和本报告。备份不包含 train/val 数据、`last.pt`、Docker 镜像、量化产物或 FPGA 产物。

## 8. 代码与环境状态

截至本报告合并时，训练仓库 HEAD 仍为 `dd70691faa8eb276301717d3d182b624cb653d14`。`predictor.py` 修改以及两个 Dockerfile、两个冒烟脚本仍在 H200 工作树中，尚未提交或推送：

```text
M  src/Yolo_fine/ultralytics/ultralytics/engine/predictor.py
?? docker/Dockerfile.predictor-chfix
?? docker/Dockerfile.predictor-gray1
?? docker/smoke_predictor_gray1.py
?? docker/smoke_predictor_warmup.py
```

本轮没有修改训练数据、`best.pt`、`last.pt`、原训练镜像 `v1`、量化 API 镜像或量化平台生产服务。

## 9. 结论与下一步

本轮 6,600 张独立 test 推理链路完整通过，证明修复后的单通道 Predictor 能稳定处理 Gray1 Pose 权重。模型在多数成功匹配实例上具有较小的中心和关键点误差，但仍有 383 个漏检、249 个误检，并存在由 `didikaka`、`aipuha` 等类别主导的长尾大误差。

建议下一步从 `matched_errors.csv` 按误差降序抽取样本，并联合 `missed_gt.csv`、`false_positives.csv` 对上述问题类别进行人工复核；在完成错误归因前，不把当前结果表述为量化后或生产交付精度。
