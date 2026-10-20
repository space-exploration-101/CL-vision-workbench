# 训练、评估、量化与 FPGA 流程

训练让模型学习权重，评估检查识别效果，量化转换数值表示，FPGA 验收检查硬件是否正确实现这些计算。模型合同约定输入与输出；smoke 是小规模功能试跑，parity 是同图结果对齐。完整概念先见[基础知识](01_YOLO_Gray1与量化基础.md)。

## 1. 固定模型合同

合同约定模型输入、输出、类别顺序和硬件布局。完整接口只在[架构说明](../architecture/Pose21_Gray1网络结构与FPGA适配说明.md)及其[机器可读参数](../architecture/pose21_gray1_fpga_contract.yaml)维护；模型哈希与量化结构以[MODEL_BASELINE](../../agent/topics/MODEL_BASELINE.md)为准。本页解释流程，不另维护一份 YAML 合同。

## 2. 阶段流程

```text
数据 VERIFIED
-> 单通道训练 smoke
-> 正式训练与 val 选 best.pt
-> 独立 test 冻结 PT 基线
-> 同集 PT 与预处理对齐
-> FP16 ONNX 导出和同集 parity
-> weight-only 权重量化与同集评估
-> 完整 6,600 张回归
-> BIN/输入回环/逐层数值对比
-> FPGA 实机和发布
```

上图是完整验收目标；当前记录已完成八阶段软件交付，但完整 6,600 张量化回归和 FPGA 实机仍待验收，不能把流程图当作全部完成状态。

每一步都是门禁。上一层失败时保留现场并定位差异，不用后续阶段的结果反向掩盖问题。

## 3. 训练与评估原则

- 从 Gray1 YAML 初始化，首层必须是 1 通道。
- 训练运行冻结代码、镜像、数据 manifest、参数和权重哈希。
- test 不参与权重选择；固定 `conf=0.25`、NMS IoU `0.7`、同类 IoU `0.5` 贪心一对一匹配。
- 同时报总体、逐类、背景图和保护类别；固定阈值 precision/recall 是代理值，不等于 COCO mAP。
- 保护类别：`didikaka`、`aipuha`、`taiwan`、`taihu`、`kanjialu`。

## 4. 量化门禁

先在同一 462 张 quick evaluation 上比较 PT、FP16 和交付候选 ONNX：

1. PT 与 FP16 必须逐图对齐，shape、dtype、输出有限且非全零。
2. 量化模型必须显式记录样本 manifest、量化类型、per-channel axis、算子名单和结构门禁结果。
3. 量化模型通过快速门禁后才运行完整 test。
4. 精度失败时检查首个偏离层、量化范围和数值合同，不调整阈值制造“通过”。

当前结构由[模型基线](../../agent/topics/MODEL_BASELINE.md)维护，09.20 的软件交付结果及旧 activation-QDQ 失败由[完整报告](../reports/current/20260920_Pose21_Gray1_训练测试标定与量化完整报告.md)解释。流程门禁不能当作已通过状态；完整量化回归与真实 FPGA 验收仍须分别取得证据。

运行依赖见[平台 02](../../platform/02_H200训练测试推理与量化运行环境.md)；备份镜像身份见[平台 01](../../platform/01_NAS备份与环境恢复.md)。

## 5. FPGA 接口门禁

- Gray1 第一层只有一个有效输入通道；32-lane 中的位置与补零规则仍需 RTL/驱动确认。
- 31 个逻辑输出通道物理补齐到 32，输出缓冲至少 `34000 x 32 x 2 = 2,176,000` bytes。
- 必须固定 INT8 权重的 signedness、scale 轴、激活和累加器位宽、zero-point、rounding 与 saturation。
- 加载 manifest 绑定 PT、ONNX、BIN、类别表和 ABI 版本，并拒绝旧 2 类/三通道资产混入。

未决项与逐层布局以[网络结构与 FPGA 适配说明](../architecture/Pose21_Gray1网络结构与FPGA适配说明.md)为准。

## 6. 运行安全

GPU 任务只在持续采样确认空闲且获得授权后启动；不停止其他用户进程。CPU 标定、cam2000 和归档不应占用 GPU。任何 OOM、NaN/Inf、通道/类别/shape 错误、磁盘不足、资源竞争或 manifest 不一致都应停止本方任务并保留现场。

## 7. Gray1 源码适配与构建回归

| 环节 | 关键处理与文件 |
|---|---|
| 图像语义 | R-only 源图显式提取 R，不能做普通加权灰度转换；net1280 为真正单通道 |
| 模型与训练 | YAML 的 ch=1，dataset.yaml 的 channels=1/input_semantics=gray1；nn/tasks.py 的 stride 初始化用实际通道，data/base.py 与 data/augment.py 保持单通道、连续数组；训练入口检查首层与配置 |
| 独立推理 | predictor.py 预热按模型通道数构造张量，恢复 letterbox 后被压掉的通道维；仅三通道做 RGB/BGR 翻转，再转 BCHW |
| 导出与评估 | net1280 不重复 resize/padding；按约定 `/255`、NCHW、FP16，ONNX 输入 `[1,1,1280,1280]`、业务输出 `[1,31,34000]` |

训练侧文件相对 `src/Yolo_fine/ultralytics/ultralytics/`，模型配置在 `src/Yolo_fine/configs/`。当前日期镜像包含训练侧适配和 Predictor 修复；只改单个文件不能把任意 RGB 模型转成当前 Gray1 模型。旧三通道权重不能靠改 YAML 直接替换单通道权重。
