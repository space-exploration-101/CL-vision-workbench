# dataset3～dataset8 地月检测与关键点实验笔记

记录日期：2026-10-25（Asia/Shanghai）。生成时间：2026-10-25T18:26:17+08:00。

工程根目录：`/data1/chenshiwen/landmarks_detect/landmarks2_new_em_v8.3.165`。

本文整理已经完成的实验、实测结果和结论。表格由保存的JSON、CSV及args.yaml读取生成；本次整理没有启动训练或追加推理。权重和图片入口列在文末。

## 1. 当前结论与适用范围

1. **负样本微调明显减少地球误检。** 完整真实集上，dataset3第200轮地球FP为730，dataset4 best降至93；排除Galileo后，地球FP从557降至59。加入星空和航天器背景对当前问题有效。
2. **按固定第20轮比较，30%负样本组dataset7的检测mAP最高。** 完整真实集mAP50–95为88.71%，非Galileo为96.91%。它不是所有类别、所有关键点指标都最优，不能据单次实验断言30%为普遍最优比例。
3. **各组保存的best.pt中，dataset7 best（微调第1轮）检测mAP最高。** 完整真实集91.13%，非Galileo97.76%；非Galileo月球P/R为96.00%/97.92%。地球关键点坐标差高于dataset4 best。
4. **dataset4 best（微调第4轮）是较均衡的候选。** 非Galileo地球P/R为95.31%/98.04%，K1/K2均值3.573/3.457px；月球为93.41%/97.05%，K1/K2均值1.515/1.522px。
5. **dataset8 last在各自匹配样本上的四项关键点均值最低，但检测有退步。** 非Galileo地球K1/K2为3.532/3.375px，月球1.479/1.463px；对应地球P=87.64%、月球R=94.28%。与dataset4 best的定位均值差距较小，需同时看共同匹配目标和误差尾部。
6. **大地球尺寸重采样没有解决Galileo问题。** dataset8相较dataset4没有带来稳定的Galileo定位改善，也未超过dataset4在合成大地球测试上的精度。不能把Galileo失败归因于“纹理丰富”或“尺寸大”这一个因素。

“best”按合成验证集训练指标保存；“真实集效果最好”还取决于检测、定位、误报、漏检和来源分布。下文同时保留best和last，主控变量比较采用相同微调第20轮。

## 2. 实验背景与已验证的问题

原始dataset3模型在合成验证集上接近指标上限，但在真实图片上有地球背景误检、月球漏检，以及Galileo地球定位偏差。背景消融使用dataset3第43轮模型和4张ISS月球图片：

| 输入处理 | 地球预测框 | 月球预测框 |
|---|---:|---:|
| 原始4张图 | 8 | 4 |
| 保留月球，背景置零 | 0 | 4 |
| 去掉月球，保留背景 | 4 | 0 |
| 仅保留均匀灰色照片矩形 | 4 | 0 |
| 整幅均匀灰色、没有月球 | 4 | 0 |
| 原图黑色padding填成灰色 | 1 | 4 |

这些消融支持“模型利用背景灰度、照片边界等捷径”这一解释；范围仅限该模型与这4张图片，不能直接外推为所有后续权重的唯一错误机制。提高置信度或调整NMS也无法从根本上清除其中的高置信度误报。

证据：[背景消融汇总](dataset3_false_earth_diagnosis/background_ablation_summary.json)、[逐图计数](dataset3_false_earth_diagnosis/background_ablation_counts.csv)。

## 3. 数据集设计

### 3.1 dataset3：基础合成正样本训练集

共40000张1024×1024图片，每图1个地球或月球，原图存储RGBA，训练读取为单通道灰度。train/val/test分别32000/4000/4000，每个划分中地球与月球各占一半，无独立纯背景图片。

4000个场景组、每组10种外观变体；保持场景组隔离。30组地球云观测按20/5/5划分，场景组、云观测和图片哈希未发现跨划分泄漏。原始表面纹理资产有意共享，因此这是合成场景分组留出，不等于新的独立真实域测试。

Earth来源为`earth_camera_20000_v10_observed_clouds`，Moon来源为`moon_camera_20000_v8_4_realstyle`。本地已校验40000对图片/标签哈希、尺寸和格式。

### 3.2 dataset4～dataset8：微调数据

负样本正式来源为上传的`data/data4/`：5000张星空、6000张航天器，共11000张，其中原始train有8800张；中间渲染产物、备份和试验图片不混入正式池。负样本保留空YOLO标签，不增加background类别。

负样本比例定义为：`负样本图片数 / train全部图片数`，不是“负样本/正样本”。

| 数据集 | train地球 | train月球 | train负样本 | 负样本占比 | train总数 | val / test | 实验因素 |
|---|---|---|---|---|---|---|---|
| dataset3 | 16000 | 16000 | 0 | 0% | 32000 | 4000 / 4000 | 基础合成训练 |
| dataset4 | 2250 | 2250 | 500 | 10.0000% | 5000 | 1000 / 1000 | 负样本比例 |
| dataset5 | 2250 | 2250 | 237 | 5.0032% | 4737 | 1000 / 1000 | 负样本比例 |
| dataset6 | 2250 | 2250 | 1125 | 20.0000% | 5625 | 1000 / 1000 | 负样本比例 |
| dataset7 | 2250 | 2250 | 1929 | 30.0047% | 6429 | 1000 / 1000 | 负样本比例 |
| dataset8 | 2250 | 2250 | 500 | 10.0000% | 5000 | 1000 / 1000 | 大地球重采样 |

dataset4～dataset8的val/test各1000张，逐图与dataset4一致：450张地球、450张月球、50张星空、50张航天器。没有按模型分数挑选验证/测试背景。

负样本比例实验dataset4/5/6/7的4500张正样本完全相同；训练负样本按来源、场景及父模型误报状态分层，保持大致一半星空、一半航天器，并采用嵌套选择：

`dataset5的237张 ⊂ dataset4的500张 ⊂ dataset6的1125张 ⊂ dataset7的1929张`。

对应星空/航天器分别为dataset4的250/250、dataset5的118/119、dataset6的562/563、dataset7的964/965。dataset4的500张训练背景中，440张会被父模型误报；该结果来自仅对原train背景筛选，不使用held-out背景分数选样。

dataset4～dataset8保持来源划分、场景组、云观测及星空种子的跨划分隔离。图片和标签为独立文件复制，像素与标签哈希保持原样；没有引入dataset2真实评估图参与训练。

### 3.3 相位与大地球专项设计

五组微调的训练相位数量一致。负样本比例组保持完整正样本一致；dataset8只在地球内部重采样，月球和500张负样本与dataset4完全一致。

| 类别 | 满相 | 接近满相 | 凸相 | 半相 | 月牙/残相 | 合计 |
|---|---|---|---|---|---|---|
| 地球 | 450 | 450 | 450 | 600 | 300 | 2250 |
| 月球 | 650 | 500 | 350 | 450 | 300 | 2250 |

大地球统一定义为：当前1024画布上GT框**宽和高都≥640px**。按此定义逐标签计数，dataset4训练地球中有356张大地球（15.82%）；dataset8为1800张大地球+450张其他地球（大地球占地球正样本80%）。不要把其他尺寸分箱名称直接当成640px阈值。

dataset8另有大地球诊断val375张、test298张，均来自原dataset3对应划分，与训练隔离；与共同val/test在同一划分内有部分重叠，不能视为完全独立的另一批测试。

Galileo的348个地球目标中只有102个满足上述640px条件，中位短边约566.75px。因此并非所有Galileo地球都是该定义下的大地球。dataset8只是对父模型见过的合成样本重新加权，没有新增真实纹理或新的纹理资产，不能单独检验“真实纹理过于丰富”这一假设。

数据制作依据：[dataset3本地审计](../dataset3/local_validation.json)、[划分审计](../dataset3/leakage_report.json)、[dataset4制作记录](../dataset4/preparation.json)、[dataset5～8方案](dataset5_8_experiments/dataset_plan.json)。

## 4. 训练配置与训练记录

所有模型为本地Ultralytics 8.3.165的YOLOv8x-pose-p6，2类、每目标2个关键点。dataset3使用官方预训练权重；dataset4～dataset8均独立从dataset3第200轮last.pt初始化，重建微调优化器，不是按dataset编号串接权重，也不是恢复父训练的优化器状态。

| 配置项 | dataset3基础训练 | dataset4～dataset8微调 |
|---|---|---|
| 初始化 | weights/yolov8x-pose-p6.pt | 同一dataset3第200轮last.pt |
| 输入 | 1024×1024、单通道灰度 | 1024×1024、单通道灰度 |
| GPU | 2,3 | 0,1 |
| 单卡 / 总batch | 64 / 128 | 64 / 128 |
| workers参数 | 24 | 24 |
| 轮数 | 200 | 20 |
| 优化器 | auto实际选择SGD | 显式SGD |
| 实际初始lr | 0.01 | 0.0005 |
| 实际momentum | 0.9（以日志为准） | 0.9 |
| lrf | 0.01 | 0.1 |
| weight_decay参数 | 0.0005 | 0.0005 |
| warmup_epochs | 3 | 1 |
| warmup_bias_lr | 0.1 | 0 |
| 关闭Mosaic | 最后10轮 | 最后5轮（第16轮开始关闭） |
| 保存周期 | 20轮 | 5轮 |
| seed | 20261014 | 20261024 |
| AMP / deterministic | 开启 / 开启 | 开启 / 开启 |
| patience | 0 | 0 |

日志显示，由于总batch128、nbs64，实际优化器权重衰减组被缩放为0.001；args.yaml中0.0005是输入配置。dataset3的args.yaml保留optimizer=auto和momentum=0.937，但训练日志实际选择SGD、momentum=0.9，记录时以实际优化器日志为准。

保持默认几何增强：Mosaic=1、translate=0.1、scale=0.5、fliplr=0.5；degrees=0、flipud=0、mixup=0。单通道分支跳过HSV变换，不能把YAML内HSV参数理解为灰度输入实际执行了颜色增强。Mosaic会缩小/裁剪大目标，dataset8保持同一增强配置以控制变量，不能假设训练中所有大地球始终保持原始尺寸。

训练损失增益相同：box=7.5、pose=12、kobj=1、cls=0.5、dfl=1.5。纯负样本主要约束分类/背景分数，不直接提供框和关键点坐标监督。pose_loss为归一化损失，不能直接当成像素误差。

| 实验 | 完成轮数 | best轮次 | last轮次 | CSV累计秒 | 累计分钟 | 配置 | 训练曲线原始值 |
|---|---|---|---|---|---|---|---|
| dataset3 | 200 | 43 | 200 | 56806.40 | 946.77 | [args](../runs/earth_moon_yolov8x_pose_p6_gray_dataset3_pretrained_gpu23_b64_200epochs/args.yaml) | [CSV](../runs/earth_moon_yolov8x_pose_p6_gray_dataset3_pretrained_gpu23_b64_200epochs/results.csv) |
| dataset4 | 20 | 4 | 20 | 1283.27 | 21.39 | [args](../runs/earth_moon_yolov8x_pose_p6_gray_dataset4_finetune_gpu01_b64_20epochs/args.yaml) | [CSV](../runs/earth_moon_yolov8x_pose_p6_gray_dataset4_finetune_gpu01_b64_20epochs/results.csv) |
| dataset5 | 20 | 11 | 20 | 1118.47 | 18.64 | [args](../runs/earth_moon_yolov8x_pose_p6_gray_dataset5_neg5_gpu01_b64_20epochs/args.yaml) | [CSV](../runs/earth_moon_yolov8x_pose_p6_gray_dataset5_neg5_gpu01_b64_20epochs/results.csv) |
| dataset6 | 20 | 5 | 20 | 1299.21 | 21.65 | [args](../runs/earth_moon_yolov8x_pose_p6_gray_dataset6_neg20_gpu01_b64_20epochs/args.yaml) | [CSV](../runs/earth_moon_yolov8x_pose_p6_gray_dataset6_neg20_gpu01_b64_20epochs/results.csv) |
| dataset7 | 20 | 1 | 20 | 1447.74 | 24.13 | [args](../runs/earth_moon_yolov8x_pose_p6_gray_dataset7_neg30_gpu01_b64_20epochs/args.yaml) | [CSV](../runs/earth_moon_yolov8x_pose_p6_gray_dataset7_neg30_gpu01_b64_20epochs/results.csv) |
| dataset8 | 20 | 4 | 20 | 1170.32 | 19.51 | [args](../runs/earth_moon_yolov8x_pose_p6_gray_dataset8_large_earth80_neg10_gpu01_b64_20epochs/args.yaml) | [CSV](../runs/earth_moon_yolov8x_pose_p6_gray_dataset8_large_earth80_neg10_gpu01_b64_20epochs/results.csv) |

上表时间为results.csv最后一行的训练累计时间，不包含所有进程启动、最终验证及外部测试开销。六个训练CSV的全部数值已检查为有限值，未发现NaN/Inf记录；这不能排除未记录到CSV的瞬时异常。

- dataset3：2026-10-18开始，10-19完成200轮；原训练耗时约15小时47分钟，另有最终验证等开销。
- dataset4：2026-10-24完成20轮微调。
- dataset5～8队列：2026-10-25 13:46:15启动，15:37:40结束（含训练及原始测试），约1小时51分钟。
- 全部六组非Galileo评估于2026-10-25完成；并行汇总流程结束于16:23:18。dataset4 best复用此前同权重、同参数、同2360张图片的已完成测试，并校验SHA及图片集合。

同样训练20轮保证每个保留正样本的遍历轮数一致，但负样本增加也增加每轮batch数和总优化步数。该实验不等于固定算力、固定更新次数的纯比例实验；dataset4～8都只做了一个随机种子，尚未测跨种子波动。

## 5. 评估口径与数据版本

真实测试使用当前dataset2全部划分，而不是仅images/test：train2192、val274、test270，共2736张，地球1572个、月球1331个。原有750张全幅地球已缩小一半并居中padding；沿用当前标注坐标，不逆映射。历史已删除4张无效测试图，当前版本沿用这一状态。

按dataset2/split_manifest.json中的spacecraft字段排除全部Galileo：376张图（地球348个、月球178个），保留2360张（地球1224个、月球1153个）。非Galileo集合覆盖train1894、val250、test216张；只在评估清单中过滤，未删除原图。

| 项目 | 口径 |
|---|---|
| 推理 | 灰度1024、FP32、batch4、rect=False、无测试增强 |
| mAP | 在对应图片集合重新运行标准box评估，候选conf=0.001、NMS IoU=0.7 |
| 固定阈值计数 | 保存预测conf=0.25；同类框Hungarian一对一匹配、IoU≥0.5 |
| P / R | 本笔记计数表为TP/(TP+FP)、TP/(TP+FN)，均固定conf=0.25 |
| 标准metrics.json中的P/R | Ultralytics最佳平均F1工作点，与固定0.25计数表不同 |
| 关键点 | 欧氏像素距离；仅匹配目标中GT visibility>0的点，包含v=1、v=2 |
| 预测关键点阈值 | 0.5仅控制画图；计算误差不筛选预测关键点置信度 |
| 漏检 | 单独计FN，不把缺失的关键点误差记为0 |
| 重复框 | 一个GT最多匹配一个预测，多余框可计为FP |

训练数据dataset3～8：K1=limb_center（完整参考轮廓椭圆中心），K2=body_center_projection（三维天体中心投影）。真实dataset2：K1=disk_center（拟合圆盘中心），K2=box_center（框中心）。因此真实集统计是**跨定义坐标差**，不是严格同定义定位精度；同定义合成集用于辅助判断。

“排除后分数升高”包含测试样本难度改变，不能解释为模型经过了新的改进。dataset2已反复用于模型比较，因此这些成绩不是未触碰的独立最终泛化成绩。

## 6. 完整真实集结果（2736张，含Galileo）

dataset3第43轮此前仅对原270张test做过评估，本次未补做其2736张全量测试，因此下表保留11份已有全量结果，不以270张指标填补2736张结果。dataset3第200轮是各微调模型的共同父模型。

### 6.1 检测mAP

| 权重 | 轮次 | 总体mAP50 | 总体mAP50–95 | 地球mAP50–95 | 月球mAP50–95 |
|---|---|---|---|---|---|
| dataset3_last | 200 | 88.34% | 84.73% | 81.41% | 88.05% |
| dataset4_best | 4 | 91.89% | 87.52% | 86.88% | 88.15% |
| dataset4_last | 20 | 92.27% | 87.76% | 87.40% | 88.12% |
| dataset5_best | 11 | 89.16% | 84.70% | 82.91% | 86.49% |
| dataset5_last | 20 | 92.15% | 87.63% | 86.91% | 88.36% |
| dataset6_best | 5 | 90.92% | 87.08% | 86.95% | 87.21% |
| dataset6_last | 20 | 92.36% | 88.32% | 88.03% | 88.60% |
| dataset7_best | 1 | 94.29% | 91.13% | 92.03% | 90.23% |
| dataset7_last | 20 | 92.78% | 88.71% | 88.72% | 88.71% |
| dataset8_best | 4 | 90.66% | 86.36% | 85.54% | 87.18% |
| dataset8_last | 20 | 91.84% | 87.52% | 86.86% | 88.19% |

### 6.2 固定conf0.25检测计数

| 权重 | 类别 | GT | TP | FP | FN | P | R |
|---|---|---|---|---|---|---|---|
| dataset3_last | earth | 1572 | 1379 | 730 | 193 | 65.39% | 87.72% |
| dataset3_last | moon | 1331 | 1117 | 101 | 214 | 91.71% | 83.92% |
| dataset4_best | earth | 1572 | 1464 | 93 | 108 | 94.03% | 93.13% |
| dataset4_best | moon | 1331 | 1147 | 79 | 184 | 93.56% | 86.18% |
| dataset4_last | earth | 1572 | 1501 | 185 | 71 | 89.03% | 95.48% |
| dataset4_last | moon | 1331 | 1112 | 43 | 219 | 96.28% | 83.55% |
| dataset5_best | earth | 1572 | 1429 | 133 | 143 | 91.49% | 90.90% |
| dataset5_best | moon | 1331 | 1130 | 78 | 201 | 93.54% | 84.90% |
| dataset5_last | earth | 1572 | 1487 | 185 | 85 | 88.94% | 94.59% |
| dataset5_last | moon | 1331 | 1120 | 84 | 211 | 93.02% | 84.15% |
| dataset6_best | earth | 1572 | 1444 | 123 | 128 | 92.15% | 91.86% |
| dataset6_best | moon | 1331 | 1137 | 76 | 194 | 93.73% | 85.42% |
| dataset6_last | earth | 1572 | 1486 | 188 | 86 | 88.77% | 94.53% |
| dataset6_last | moon | 1331 | 1116 | 44 | 215 | 96.21% | 83.85% |
| dataset7_best | earth | 1572 | 1524 | 120 | 48 | 92.70% | 96.95% |
| dataset7_best | moon | 1331 | 1157 | 47 | 174 | 96.10% | 86.93% |
| dataset7_last | earth | 1572 | 1496 | 187 | 76 | 88.89% | 95.17% |
| dataset7_last | moon | 1331 | 1123 | 37 | 208 | 96.81% | 84.37% |
| dataset8_best | earth | 1572 | 1441 | 93 | 131 | 93.94% | 91.67% |
| dataset8_best | moon | 1331 | 1151 | 78 | 180 | 93.65% | 86.48% |
| dataset8_last | earth | 1572 | 1474 | 200 | 98 | 88.05% | 93.77% |
| dataset8_last | moon | 1331 | 1110 | 57 | 221 | 95.12% | 83.40% |

### 6.3 真实关键点均值（px）

| 权重 | 地球K1 n | 月球K1 n | 地球K1 | 地球K2 | 月球K1 | 月球K2 |
|---|---|---|---|---|---|---|
| dataset3_last | 1379 | 1117 | 12.642 | 12.465 | 2.068 | 2.071 |
| dataset4_best | 1464 | 1147 | 10.830 | 10.662 | 1.672 | 1.679 |
| dataset4_last | 1501 | 1112 | 11.613 | 11.353 | 1.652 | 1.608 |
| dataset5_best | 1429 | 1130 | 13.140 | 12.957 | 1.815 | 1.802 |
| dataset5_last | 1487 | 1120 | 12.790 | 12.567 | 1.703 | 1.694 |
| dataset6_best | 1444 | 1137 | 9.709 | 9.565 | 1.671 | 1.659 |
| dataset6_last | 1486 | 1116 | 11.962 | 11.756 | 1.617 | 1.594 |
| dataset7_best | 1524 | 1157 | 7.583 | 7.427 | 1.757 | 1.747 |
| dataset7_last | 1496 | 1123 | 10.975 | 10.656 | 1.655 | 1.639 |
| dataset8_best | 1441 | 1151 | 10.795 | 10.674 | 1.690 | 1.732 |
| dataset8_last | 1474 | 1110 | 12.269 | 12.095 | 1.626 | 1.614 |

### 6.4 dataset3第43轮与第200轮的历史270张test比较

该270张测试与本节2736张全量集规模不同，不能混用绝对成绩。保留下来的43轮/200轮在此集合mAP50–95分别71.01%/74.18%，200轮更好。对现存11个检查点进行过比较，200轮在这些保留权重中最高；没有保留每一轮，因此不能声称遍历了1～200所有权重。

历史证据：[保留检查点排名](../test/results/dataset3_real_checkpoint_comparison/ranking.csv)。

## 7. 排除Galileo后结果（2360张）

六组best/last共12份权重已经对同一图片集合完成独立标准框评估和推理；每份结果都有2360张标注图。没有把全量mAP按数量相减来估算子集mAP。

### 7.1 检测mAP及相对完整集变化

| 权重 | 轮次 | 完整集mAP50–95 | 非Galileo mAP50–95 | 集合变化差值 | 地球mAP50–95 | 月球mAP50–95 |
|---|---|---|---|---|---|---|
| dataset3_best | 43 | 未评估全量集 | 92.03% | — | 89.90% | 94.17% |
| dataset3_last | 200 | 84.73% | 95.52% | +10.79pp | 93.76% | 97.28% |
| dataset4_best | 4 | 87.52% | 97.54% | +10.02pp | 97.03% | 98.05% |
| dataset4_last | 20 | 87.76% | 96.30% | +8.54pp | 95.80% | 96.79% |
| dataset5_best | 11 | 84.70% | 96.07% | +11.37pp | 95.70% | 96.44% |
| dataset5_last | 20 | 87.63% | 96.37% | +8.73pp | 95.77% | 96.97% |
| dataset6_best | 5 | 87.08% | 97.23% | +10.15pp | 97.35% | 97.12% |
| dataset6_last | 20 | 88.32% | 96.65% | +8.33pp | 96.24% | 97.05% |
| dataset7_best | 1 | 91.13% | 97.76% | +6.64pp | 97.09% | 98.44% |
| dataset7_last | 20 | 88.71% | 96.91% | +8.20pp | 96.51% | 97.32% |
| dataset8_best | 4 | 86.36% | 97.29% | +10.93pp | 97.29% | 97.28% |
| dataset8_last | 20 | 87.52% | 96.69% | +9.16pp | 96.43% | 96.94% |

### 7.2 固定conf0.25检测计数

| 权重 | 类别 | GT | TP | FP | FN | P | R |
|---|---|---|---|---|---|---|---|
| dataset3_best | earth | 1224 | 1146 | 851 | 78 | 57.39% | 93.63% |
| dataset3_best | moon | 1153 | 1066 | 170 | 87 | 86.25% | 92.45% |
| dataset3_last | earth | 1224 | 1170 | 557 | 54 | 67.75% | 95.59% |
| dataset3_last | moon | 1153 | 1096 | 100 | 57 | 91.64% | 95.06% |
| dataset4_best | earth | 1224 | 1200 | 59 | 24 | 95.31% | 98.04% |
| dataset4_best | moon | 1153 | 1119 | 79 | 34 | 93.41% | 97.05% |
| dataset4_last | earth | 1224 | 1201 | 151 | 23 | 88.83% | 98.12% |
| dataset4_last | moon | 1153 | 1089 | 43 | 64 | 96.20% | 94.45% |
| dataset5_best | earth | 1224 | 1204 | 73 | 20 | 94.28% | 98.37% |
| dataset5_best | moon | 1153 | 1102 | 78 | 51 | 93.39% | 95.58% |
| dataset5_last | earth | 1224 | 1198 | 138 | 26 | 89.67% | 97.88% |
| dataset5_last | moon | 1153 | 1097 | 84 | 56 | 92.89% | 95.14% |
| dataset6_best | earth | 1224 | 1202 | 88 | 22 | 93.18% | 98.20% |
| dataset6_best | moon | 1153 | 1110 | 76 | 43 | 93.59% | 96.27% |
| dataset6_last | earth | 1224 | 1200 | 150 | 24 | 88.89% | 98.04% |
| dataset6_last | moon | 1153 | 1096 | 44 | 57 | 96.14% | 95.06% |
| dataset7_best | earth | 1224 | 1198 | 84 | 26 | 93.45% | 97.88% |
| dataset7_best | moon | 1153 | 1129 | 47 | 24 | 96.00% | 97.92% |
| dataset7_last | earth | 1224 | 1197 | 150 | 27 | 88.86% | 97.79% |
| dataset7_last | moon | 1153 | 1100 | 37 | 53 | 96.75% | 95.40% |
| dataset8_best | earth | 1224 | 1198 | 55 | 26 | 95.61% | 97.88% |
| dataset8_best | moon | 1153 | 1123 | 78 | 30 | 93.51% | 97.40% |
| dataset8_last | earth | 1224 | 1198 | 169 | 26 | 87.64% | 97.88% |
| dataset8_last | moon | 1153 | 1087 | 57 | 66 | 95.02% | 94.28% |

### 7.3 非Galileo关键点均值（px）

| 权重 | 地球K1 | 地球K2 | 月球K1 | 月球K2 |
|---|---|---|---|---|
| dataset3_best | 7.389 | 7.280 | 1.938 | 1.995 |
| dataset3_last | 5.448 | 5.301 | 1.707 | 1.712 |
| dataset4_best | 3.573 | 3.457 | 1.515 | 1.522 |
| dataset4_last | 3.934 | 3.723 | 1.512 | 1.465 |
| dataset5_best | 4.310 | 4.164 | 1.611 | 1.597 |
| dataset5_last | 3.714 | 3.532 | 1.551 | 1.541 |
| dataset6_best | 3.899 | 3.786 | 1.508 | 1.493 |
| dataset6_last | 3.839 | 3.684 | 1.520 | 1.494 |
| dataset7_best | 4.872 | 4.709 | 1.649 | 1.636 |
| dataset7_last | 3.697 | 3.405 | 1.538 | 1.522 |
| dataset8_best | 3.918 | 3.820 | 1.497 | 1.537 |
| dataset8_last | 3.532 | 3.375 | 1.479 | 1.463 |

### 7.4 非Galileo关键点误差分布

| 权重 | 关键点 | n | 均值px | 中位数px | P90px | P95px | 最大px | ≤3px | ≤5px |
|---|---|---|---|---|---|---|---|---|---|
| dataset3_best | earth_K1 | 1146 | 7.389 | 5.733 | 8.509 | 13.335 | 240.054 | 24.43% | 40.75% |
| dataset3_best | earth_K2 | 1146 | 7.280 | 5.588 | 8.401 | 13.135 | 238.311 | 24.61% | 41.27% |
| dataset3_best | moon_K1 | 1066 | 1.938 | 1.437 | 3.713 | 5.073 | 38.644 | 82.55% | 94.84% |
| dataset3_best | moon_K2 | 1066 | 1.995 | 1.549 | 3.717 | 5.337 | 37.104 | 82.65% | 94.47% |
| dataset3_last | earth_K1 | 1170 | 5.448 | 3.706 | 6.310 | 8.485 | 231.936 | 32.99% | 77.52% |
| dataset3_last | earth_K2 | 1170 | 5.301 | 3.505 | 6.133 | 8.507 | 231.059 | 37.52% | 79.49% |
| dataset3_last | moon_K1 | 1096 | 1.707 | 1.314 | 2.883 | 3.521 | 54.798 | 91.15% | 97.35% |
| dataset3_last | moon_K2 | 1096 | 1.712 | 1.259 | 2.883 | 4.145 | 54.285 | 90.97% | 97.08% |
| dataset4_best | earth_K1 | 1200 | 3.573 | 3.320 | 5.638 | 6.530 | 32.734 | 39.83% | 82.25% |
| dataset4_best | earth_K2 | 1200 | 3.457 | 3.262 | 5.442 | 6.318 | 32.727 | 41.25% | 84.92% |
| dataset4_best | moon_K1 | 1119 | 1.515 | 1.059 | 3.214 | 4.217 | 12.406 | 88.29% | 97.86% |
| dataset4_best | moon_K2 | 1119 | 1.522 | 1.057 | 3.161 | 4.423 | 16.567 | 88.65% | 97.59% |
| dataset4_last | earth_K1 | 1201 | 3.934 | 3.757 | 5.580 | 6.100 | 52.997 | 31.56% | 78.68% |
| dataset4_last | earth_K2 | 1201 | 3.723 | 3.504 | 5.377 | 5.888 | 53.015 | 36.64% | 84.43% |
| dataset4_last | moon_K1 | 1089 | 1.512 | 1.063 | 3.518 | 3.900 | 11.995 | 86.96% | 97.89% |
| dataset4_last | moon_K2 | 1089 | 1.465 | 1.026 | 3.273 | 3.790 | 15.478 | 87.60% | 97.70% |
| dataset5_best | earth_K1 | 1204 | 4.310 | 4.325 | 6.133 | 6.698 | 42.152 | 22.34% | 69.35% |
| dataset5_best | earth_K2 | 1204 | 4.164 | 4.137 | 5.979 | 6.511 | 41.847 | 23.50% | 73.09% |
| dataset5_best | moon_K1 | 1102 | 1.611 | 1.040 | 3.632 | 4.913 | 14.083 | 85.57% | 95.46% |
| dataset5_best | moon_K2 | 1102 | 1.597 | 1.036 | 3.599 | 5.186 | 14.105 | 86.75% | 94.83% |
| dataset5_last | earth_K1 | 1198 | 3.714 | 3.541 | 5.692 | 6.054 | 27.645 | 37.06% | 77.88% |
| dataset5_last | earth_K2 | 1198 | 3.532 | 3.348 | 5.372 | 5.759 | 27.367 | 40.98% | 82.72% |
| dataset5_last | moon_K1 | 1097 | 1.551 | 1.089 | 3.268 | 3.811 | 11.644 | 85.14% | 97.54% |
| dataset5_last | moon_K2 | 1097 | 1.541 | 1.035 | 3.190 | 4.008 | 15.938 | 86.42% | 97.27% |
| dataset6_best | earth_K1 | 1202 | 3.899 | 3.716 | 6.035 | 6.721 | 34.766 | 33.94% | 75.46% |
| dataset6_best | earth_K2 | 1202 | 3.786 | 3.574 | 5.852 | 6.497 | 34.415 | 35.11% | 78.54% |
| dataset6_best | moon_K1 | 1110 | 1.508 | 1.090 | 3.249 | 3.677 | 13.900 | 87.48% | 97.48% |
| dataset6_best | moon_K2 | 1110 | 1.493 | 1.059 | 3.245 | 3.853 | 13.945 | 87.75% | 97.39% |
| dataset6_last | earth_K1 | 1200 | 3.839 | 3.683 | 5.906 | 6.359 | 26.690 | 33.17% | 73.75% |
| dataset6_last | earth_K2 | 1200 | 3.684 | 3.495 | 5.658 | 6.170 | 26.320 | 37.33% | 78.92% |
| dataset6_last | moon_K1 | 1096 | 1.520 | 1.082 | 3.458 | 3.846 | 12.948 | 86.31% | 97.72% |
| dataset6_last | moon_K2 | 1096 | 1.494 | 1.039 | 3.410 | 4.016 | 12.255 | 86.86% | 97.63% |
| dataset7_best | earth_K1 | 1198 | 4.872 | 5.033 | 7.571 | 8.172 | 23.423 | 24.62% | 49.50% |
| dataset7_best | earth_K2 | 1198 | 4.709 | 4.886 | 7.296 | 7.924 | 23.215 | 25.21% | 52.34% |
| dataset7_best | moon_K1 | 1129 | 1.649 | 1.129 | 3.378 | 4.348 | 12.857 | 85.30% | 96.72% |
| dataset7_best | moon_K2 | 1129 | 1.636 | 1.105 | 3.365 | 4.386 | 17.231 | 85.03% | 96.37% |
| dataset7_last | earth_K1 | 1197 | 3.697 | 3.561 | 5.506 | 6.313 | 23.351 | 30.58% | 83.29% |
| dataset7_last | earth_K2 | 1197 | 3.405 | 3.193 | 5.133 | 5.985 | 23.123 | 42.69% | 88.47% |
| dataset7_last | moon_K1 | 1100 | 1.538 | 1.036 | 3.240 | 3.921 | 9.685 | 86.09% | 97.27% |
| dataset7_last | moon_K2 | 1100 | 1.522 | 1.017 | 3.193 | 4.016 | 13.329 | 87.55% | 97.18% |
| dataset8_best | earth_K1 | 1198 | 3.918 | 3.834 | 6.039 | 6.836 | 33.651 | 29.97% | 75.21% |
| dataset8_best | earth_K2 | 1198 | 3.820 | 3.688 | 5.917 | 6.713 | 33.614 | 32.80% | 77.38% |
| dataset8_best | moon_K1 | 1123 | 1.497 | 0.963 | 3.079 | 3.885 | 16.209 | 88.87% | 97.42% |
| dataset8_best | moon_K2 | 1123 | 1.537 | 0.956 | 3.101 | 4.491 | 17.196 | 89.31% | 96.97% |
| dataset8_last | earth_K1 | 1198 | 3.532 | 3.433 | 5.603 | 6.272 | 21.843 | 40.40% | 80.88% |
| dataset8_last | earth_K2 | 1198 | 3.375 | 3.184 | 5.479 | 6.133 | 21.887 | 44.91% | 83.72% |
| dataset8_last | moon_K1 | 1087 | 1.479 | 1.024 | 3.193 | 3.744 | 15.903 | 86.38% | 97.88% |
| dataset8_last | moon_K2 | 1087 | 1.463 | 0.996 | 2.998 | 3.680 | 15.796 | 89.97% | 97.61% |

### 7.5 所有12份权重共同匹配目标

为减少“不同模型检出的目标难度不同”带来的均值偏差，另取所有模型均成功匹配的同一GT目标：1125个地球、1042个月球（每个关键点分别核对有效性），比较同一批点。该集合偏向所有模型都容易检出的目标，不能取代完整召回率。

| 权重 | 地球K1均值px | 地球K2均值px | 月球K1均值px | 月球K2均值px |
|---|---|---|---|---|
| dataset3_best | 6.845 | 6.735 | 1.905 | 1.963 |
| dataset3_last | 5.034 | 4.875 | 1.499 | 1.507 |
| dataset4_best | 3.566 | 3.440 | 1.432 | 1.438 |
| dataset4_last | 3.938 | 3.724 | 1.439 | 1.394 |
| dataset5_best | 4.313 | 4.157 | 1.525 | 1.513 |
| dataset5_last | 3.712 | 3.525 | 1.462 | 1.456 |
| dataset6_best | 3.888 | 3.766 | 1.434 | 1.422 |
| dataset6_last | 3.846 | 3.688 | 1.443 | 1.425 |
| dataset7_best | 4.970 | 4.794 | 1.562 | 1.547 |
| dataset7_last | 3.728 | 3.424 | 1.477 | 1.464 |
| dataset8_best | 3.935 | 3.822 | 1.380 | 1.418 |
| dataset8_last | 3.557 | 3.388 | 1.411 | 1.393 |

共同目标结果说明，“四项总体均值最低”不能直接理解为同一批目标上所有项也最低。例如月球K1共同目标均值，dataset8 best为1.380px，dataset8 last为1.411px；地球K1的dataset8 last与dataset4 best分别3.557/3.566px，差距很小。

## 8. 共同合成测试与纯负样本误报

使用统一1000张test，其中900张正样本、100张纯负样本（50星空、50航天器），关键点定义与模型相同。标准mAP主要已接近上限，需同时看固定conf0.25纯背景误报图数和关键点误差。

| 权重 | mAP50–95 | 星空误报图/50 | 航天器误报图/50 | 地球K1px | 地球K2px | 月球K1px | 月球K2px |
|---|---|---|---|---|---|---|---|
| dataset3_last | 99.44% | 49 | 29 | 2.661 | 2.237 | 2.071 | 1.853 |
| dataset4_best | 99.46% | 1 | 0 | 2.053 | 1.806 | 1.644 | 1.655 |
| dataset4_last | 99.50% | 0 | 0 | 1.961 | 1.792 | 1.595 | 1.596 |
| dataset5_best | 99.50% | 1 | 0 | 2.223 | 1.961 | 1.815 | 1.787 |
| dataset5_last | 99.50% | 0 | 0 | 2.050 | 1.762 | 1.642 | 1.580 |
| dataset6_best | 99.50% | 1 | 0 | 2.059 | 1.829 | 1.719 | 1.702 |
| dataset6_last | 99.50% | 0 | 0 | 1.926 | 1.780 | 1.573 | 1.595 |
| dataset7_best | 99.49% | 3 | 3 | 2.230 | 2.088 | 1.884 | 1.927 |
| dataset7_last | 99.50% | 0 | 0 | 1.941 | 1.734 | 1.606 | 1.547 |
| dataset8_best | 99.49% | 1 | 0 | 2.068 | 1.831 | 1.655 | 1.590 |
| dataset8_last | 99.50% | 0 | 0 | 2.114 | 1.762 | 1.670 | 1.534 |

dataset4～8的第20轮last在100张合成背景上均为0张误报；dataset7 best仍有3张星空和3张航天器误报。纯背景测试已无误报不等于真实正样本背景中的误报也解决，且100张样本规模较小，不能保证任意真实星空/航天器均无误报。

## 9. Galileo来源诊断

Galileo共376张图，包含348个地球、178个月球，部分图包含两个目标。下表来自完整真实集的来源统计；地球/月球坐标均值是已匹配对象的K1，遗漏对象另由TP/GT体现。

| 权重 | 地球TP/GT | 地球R | 地球K1px | 月球TP/GT | 月球R | 月球K1px |
|---|---|---|---|---|---|---|
| dataset3_last | 209/348 | 60.06% | 52.927 | 21/178 | 11.80% | 20.906 |
| dataset4_best | 264/348 | 75.86% | 43.812 | 28/178 | 15.73% | 7.923 |
| dataset4_last | 300/348 | 86.21% | 42.350 | 23/178 | 12.92% | 8.208 |
| dataset5_best | 225/348 | 64.66% | 60.391 | 28/178 | 15.73% | 9.826 |
| dataset5_last | 289/348 | 83.05% | 50.416 | 23/178 | 12.92% | 8.964 |
| dataset6_best | 242/348 | 69.54% | 38.564 | 27/178 | 15.17% | 8.387 |
| dataset6_last | 286/348 | 82.18% | 46.044 | 20/178 | 11.24% | 6.928 |
| dataset7_best | 326/348 | 93.68% | 17.544 | 28/178 | 15.73% | 6.114 |
| dataset7_last | 299/348 | 85.92% | 40.114 | 23/178 | 12.92% | 7.184 |
| dataset8_best | 243/348 | 69.83% | 44.688 | 28/178 | 15.73% | 9.409 |
| dataset8_last | 276/348 | 79.31% | 50.194 | 23/178 | 12.92% | 8.583 |

dataset7 best对Galileo地球改善最明显：相对dataset4 best，检出264/348→326/348，K1均值43.812→17.544px；但月球仍只有28/178检出，不能因为已检出月球的误差较小就认为Galileo月球问题已经解决。

dataset7 best相对dataset4 best的mAP优势，在完整真实集为约3.61个百分点，排除Galileo后仅约0.22个百分点；其总体关键点均值优势主要来自Galileo的改善，非Galileo地球K1反而3.573→4.872px。来源组成会改变模型选择结论。

## 10. 合成大地球专项（298张）

测试来自原dataset3/test中宽高均≥640px的地球，与训练隔离，关键点定义一致。以下mAP50–95均约99.50%，检测指标已接近上限；关键点仍可用于区分。

| 权重 | 地球mAP50–95 | K1均值px | K2均值px |
|---|---|---|---|
| dataset8_last / epoch20 | 99.50% | 2.468 | 2.610 |
| dataset8_best / epoch4 | 99.50% | 2.879 | 3.012 |
| dataset3_last / epoch200 | 99.50% | 4.006 | 4.058 |
| dataset4_last / epoch20 | 99.50% | 2.434 | 2.586 |

dataset8 last相对父模型有改善，但未超过已有dataset4 last；Galileo地球K1也未改善（dataset4 last 42.350px，dataset8 last 50.194px）。因此当前证据只支持“微调改善合成域定位”，不支持“提高大地球占比就能解决Galileo”。

## 11. 结论、原因解释与局限

### 11.1 已有证据支持的结论

- 背景负样本可明显降低误报，属于有效微调方向；dataset4已带来主要增益。
- 固定第20轮时，dataset7（30%）在完整真实集和非Galileo集的检测mAP均最高；增加负样本比例并未让每项误检/定位指标单调改善。
- best.pt与last.pt存在明显取舍。dataset7第1轮best检测更好，第20轮last地球关键点更好、合成背景误报更少。不能仅根据“轮数更多”或“文件叫best”判断最终使用权重。
- dataset3的43轮best属于合成验证集最优，真实270张test及非Galileo2360张上200轮last更好。合成验证集与真实任务排序并不一致。
- Galileo是当前严重误差和月球漏检的重要来源；去掉它能提高报表成绩，但不会自动修复模型。

### 11.2 尚未被证实的原因

- Galileo差异可能涉及纹理、光照/相位、对比度、成像噪声、尺度、光学外观和标注定义。现有实验没有将这些因素逐一隔离，不能把错误全部归因于纹理丰富。
- dataset8没有新增真实纹理，只改变已有样本的尺寸分布；它没有改善Galileo，不表示任何大目标专项方法都无效。
- 不能只凭best早于last、真实指标波动就认定训练整体过拟合。需要同口径曲线、更多保留中间权重、重复种子和新真实集验证。
- 关键点跨定义、条件于匹配对象，以及来源内连续帧的相关性，都会影响均值与表面样本量；本记录未做按来源组bootstrap或显著性检验。

### 11.3 使用建议

| 使用目标 | 当前候选 | 需接受的取舍 |
|---|---|---|
| 真实集整体检测、月球检出、Galileo地球检出 | dataset7 best（第1轮） | 非Galileo地球关键点更差；100张合成背景有6张误报 |
| 地球定位并兼顾误报与召回 | dataset4 best（第4轮） | 真实整体检测mAP略低于dataset7 best，Galileo仍困难 |
| 比较非Galileo关键点均值 | dataset8 last（第20轮） | 定位增益很小；地球误报和月球漏检更多，不能仅凭均值部署 |
| 固定20轮的负样本比例基线 | dataset7 last（第20轮） | 比较主效应适用，但不是所有定位项最佳 |

### 11.4 后续实验建议（尚未执行）

1. 优先保持统一评估脚本与来源分层，另准备未参与模型选择的新真实留出集。
2. 对关键候选dataset4/7/8增加重复种子；负样本比例另补固定优化步数或固定正负采样权重对照，区分比例与更新次数因素。
3. 正样本同步加入灰色背景、照片边界、杂散光、噪声、模糊及对比度变化，避免只靠负样本学习背景。
4. 若补充真实正样本，先统一K1/K2标注定义，并按卫星/连续序列划分，避免将当前真实评估图回流后继续当独立测试。
5. 针对Galileo月球严重漏检单独分析尺寸、相位、对比度和阈值；不能只统计已检出对象的坐标误差。
6. 大地球因素继续采用固定相位、背景和纹理的受控设计；新的真实纹理/成像风格应作为独立变量，不与负样本比例同时改变。

## 12. 权重、结果与原始证据索引

### 12.1 权重及SHA-256

轮次来自checkpoint指标与训练历史匹配恢复；训练完成后的strip可能把原始epoch字段改成-1，不能据此解释为第0轮。下表完整SHA用于核验同一权重。

| 权重 | 轮次 | 文件 | SHA-256 |
|---|---|---|---|
| dataset3_best | 43 | [best.pt](../runs/earth_moon_yolov8x_pose_p6_gray_dataset3_pretrained_gpu23_b64_200epochs/weights/best.pt) | `585af1dcc0487dd71e1949505af5e88da08bce6ca736555007b5f209e5468c93` |
| dataset3_last | 200 | [last.pt](../runs/earth_moon_yolov8x_pose_p6_gray_dataset3_pretrained_gpu23_b64_200epochs/weights/last.pt) | `8b67c698ec27c148f979a9bc2a4cf26d179462127a3ce430eaf574a1f15df3c2` |
| dataset4_best | 4 | [best.pt](../runs/earth_moon_yolov8x_pose_p6_gray_dataset4_finetune_gpu01_b64_20epochs/weights/best.pt) | `6dbbae1597c261a905d060677441e5d84bed6bfcc2adb6ded45362bb56d33124` |
| dataset4_last | 20 | [last.pt](../runs/earth_moon_yolov8x_pose_p6_gray_dataset4_finetune_gpu01_b64_20epochs/weights/last.pt) | `0b1b16d64b62a331a89d8e025c76272fbc31d02443b720a96afad06eb153ec18` |
| dataset5_best | 11 | [best.pt](../runs/earth_moon_yolov8x_pose_p6_gray_dataset5_neg5_gpu01_b64_20epochs/weights/best.pt) | `378bf39e24871f99b747965e86365e58b462ce0afc947197fa8a63f124d3d340` |
| dataset5_last | 20 | [last.pt](../runs/earth_moon_yolov8x_pose_p6_gray_dataset5_neg5_gpu01_b64_20epochs/weights/last.pt) | `f511b692b3e7fb3c7cbfd8ddab0982fc9a3c5c30c130fdebd56df2a60449d36d` |
| dataset6_best | 5 | [best.pt](../runs/earth_moon_yolov8x_pose_p6_gray_dataset6_neg20_gpu01_b64_20epochs/weights/best.pt) | `c1da337ebc0ec38bd96513f19cc1371171bedb10864060de5d95260bbcb37015` |
| dataset6_last | 20 | [last.pt](../runs/earth_moon_yolov8x_pose_p6_gray_dataset6_neg20_gpu01_b64_20epochs/weights/last.pt) | `05e29f187f7ddc29dde0c49b473818e260507a41df2876b12d42c6cad2e166f7` |
| dataset7_best | 1 | [best.pt](../runs/earth_moon_yolov8x_pose_p6_gray_dataset7_neg30_gpu01_b64_20epochs/weights/best.pt) | `cad493746e68952d757ad5646402f6c7ddef1634be718883bac9714c1e9c73b2` |
| dataset7_last | 20 | [last.pt](../runs/earth_moon_yolov8x_pose_p6_gray_dataset7_neg30_gpu01_b64_20epochs/weights/last.pt) | `2bef80c1df4a9f7f628e02c483c962cff19736161bc0e5a4056693573f50177d` |
| dataset8_best | 4 | [best.pt](../runs/earth_moon_yolov8x_pose_p6_gray_dataset8_large_earth80_neg10_gpu01_b64_20epochs/weights/best.pt) | `6c2af6a907ccc0f705fe80408519b255a906505c166fd460b3af9dea238b36fa` |
| dataset8_last | 20 | [last.pt](../runs/earth_moon_yolov8x_pose_p6_gray_dataset8_large_earth80_neg10_gpu01_b64_20epochs/weights/last.pt) | `f905da48199405c750f7b93fd82328a0549547f828c5b75225d91c17d9588fe6` |

### 12.2 每份权重的推理图片和误差分布

| 权重 | 完整真实集2736张 | 非Galileo 2360张 | 共同合成test1000张 |
|---|---|---|---|
| dataset3_best | 未做2736张全量测试 | [图片](../test/results/dataset3_best_585af1dc_on_dataset2_excluding_galileo/annotated) / [误差分布](../test/results/dataset3_best_585af1dc_on_dataset2_excluding_galileo/keypoint_analysis) | 未纳入共同合成测试 |
| dataset3_last | [图片](../test/results/dataset3_last_epoch200_on_dataset2_all/annotated) / [误差](../test/results/dataset3_last_epoch200_on_dataset2_all/keypoint_coordinate_discrepancies) | [图片](../test/results/dataset3_last_8b67c698_on_dataset2_excluding_galileo/annotated) / [误差分布](../test/results/dataset3_last_8b67c698_on_dataset2_excluding_galileo/keypoint_analysis) | [合成测试图片](../test/results/dataset4_parent_epoch200_on_dataset4_test/annotated) |
| dataset4_best | [图片](../test/results/dataset4_finetune_best_on_dataset2_all/annotated) / [误差](../test/results/dataset4_finetune_best_on_dataset2_all/keypoint_analysis) | [图片](../test/results/dataset4_finetune_best_on_dataset2_excluding_galileo/annotated) / [误差分布](../test/results/dataset4_finetune_best_on_dataset2_excluding_galileo/keypoint_analysis) | [合成测试图片](../test/results/dataset4_finetune_best_on_dataset4_test/annotated) |
| dataset4_last | [图片](../test/results/dataset4_finetune_last_on_dataset2_all/annotated) / [误差](../test/results/dataset4_finetune_last_on_dataset2_all/keypoint_analysis) | [图片](../test/results/dataset4_last_0b1b16d6_on_dataset2_excluding_galileo/annotated) / [误差分布](../test/results/dataset4_last_0b1b16d6_on_dataset2_excluding_galileo/keypoint_analysis) | [合成测试图片](../test/results/dataset4_finetune_last_on_dataset4_test/annotated) |
| dataset5_best | [图片](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset5_neg5_gpu01_b64_20epochs_best_378bf39e_real_all/annotated) / [误差](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset5_neg5_gpu01_b64_20epochs_best_378bf39e_real_all/keypoint_analysis) | [图片](../test/results/dataset5_best_378bf39e_on_dataset2_excluding_galileo/annotated) / [误差分布](../test/results/dataset5_best_378bf39e_on_dataset2_excluding_galileo/keypoint_analysis) | [合成测试图片](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset5_neg5_gpu01_b64_20epochs_best_378bf39e_common_test/annotated) |
| dataset5_last | [图片](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset5_neg5_gpu01_b64_20epochs_last_f511b692_real_all/annotated) / [误差](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset5_neg5_gpu01_b64_20epochs_last_f511b692_real_all/keypoint_analysis) | [图片](../test/results/dataset5_last_f511b692_on_dataset2_excluding_galileo/annotated) / [误差分布](../test/results/dataset5_last_f511b692_on_dataset2_excluding_galileo/keypoint_analysis) | [合成测试图片](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset5_neg5_gpu01_b64_20epochs_last_f511b692_common_test/annotated) |
| dataset6_best | [图片](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset6_neg20_gpu01_b64_20epochs_best_c1da337e_real_all/annotated) / [误差](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset6_neg20_gpu01_b64_20epochs_best_c1da337e_real_all/keypoint_analysis) | [图片](../test/results/dataset6_best_c1da337e_on_dataset2_excluding_galileo/annotated) / [误差分布](../test/results/dataset6_best_c1da337e_on_dataset2_excluding_galileo/keypoint_analysis) | [合成测试图片](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset6_neg20_gpu01_b64_20epochs_best_c1da337e_common_test/annotated) |
| dataset6_last | [图片](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset6_neg20_gpu01_b64_20epochs_last_05e29f18_real_all/annotated) / [误差](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset6_neg20_gpu01_b64_20epochs_last_05e29f18_real_all/keypoint_analysis) | [图片](../test/results/dataset6_last_05e29f18_on_dataset2_excluding_galileo/annotated) / [误差分布](../test/results/dataset6_last_05e29f18_on_dataset2_excluding_galileo/keypoint_analysis) | [合成测试图片](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset6_neg20_gpu01_b64_20epochs_last_05e29f18_common_test/annotated) |
| dataset7_best | [图片](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset7_neg30_gpu01_b64_20epochs_best_cad49374_real_all/annotated) / [误差](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset7_neg30_gpu01_b64_20epochs_best_cad49374_real_all/keypoint_analysis) | [图片](../test/results/dataset7_best_cad49374_on_dataset2_excluding_galileo/annotated) / [误差分布](../test/results/dataset7_best_cad49374_on_dataset2_excluding_galileo/keypoint_analysis) | [合成测试图片](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset7_neg30_gpu01_b64_20epochs_best_cad49374_common_test/annotated) |
| dataset7_last | [图片](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset7_neg30_gpu01_b64_20epochs_last_2bef80c1_real_all/annotated) / [误差](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset7_neg30_gpu01_b64_20epochs_last_2bef80c1_real_all/keypoint_analysis) | [图片](../test/results/dataset7_last_2bef80c1_on_dataset2_excluding_galileo/annotated) / [误差分布](../test/results/dataset7_last_2bef80c1_on_dataset2_excluding_galileo/keypoint_analysis) | [合成测试图片](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset7_neg30_gpu01_b64_20epochs_last_2bef80c1_common_test/annotated) |
| dataset8_best | [图片](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset8_large_earth80_neg10_gpu01_b64_20epochs_best_6c2af6a9_real_all/annotated) / [误差](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset8_large_earth80_neg10_gpu01_b64_20epochs_best_6c2af6a9_real_all/keypoint_analysis) | [图片](../test/results/dataset8_best_6c2af6a9_on_dataset2_excluding_galileo/annotated) / [误差分布](../test/results/dataset8_best_6c2af6a9_on_dataset2_excluding_galileo/keypoint_analysis) | [合成测试图片](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset8_large_earth80_neg10_gpu01_b64_20epochs_best_6c2af6a9_common_test/annotated) |
| dataset8_last | [图片](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset8_large_earth80_neg10_gpu01_b64_20epochs_last_f905da48_real_all/annotated) / [误差](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset8_large_earth80_neg10_gpu01_b64_20epochs_last_f905da48_real_all/keypoint_analysis) | [图片](../test/results/dataset8_last_f905da48_on_dataset2_excluding_galileo/annotated) / [误差分布](../test/results/dataset8_last_f905da48_on_dataset2_excluding_galileo/keypoint_analysis) | [合成测试图片](../test/results/earth_moon_yolov8x_pose_p6_gray_dataset8_large_earth80_neg10_gpu01_b64_20epochs_last_f905da48_common_test/annotated) |

### 12.3 汇总、数据审计及训练日志

- [dataset5～8原始实测汇总](dataset5_8_experiments/results.md)

- [dataset5～8完整JSON统计](dataset5_8_experiments/results_summary.json)

- [dataset4微调前后对比](../test/results/dataset4_finetune_comparison/README.md)

- [非Galileo十二份权重完整报告](../test/results/dataset3_8_excluding_galileo_comparison/README.md)

- [非Galileo完整JSON](../test/results/dataset3_8_excluding_galileo_comparison/summary.json)

- [非Galileo扁平统计CSV](../test/results/dataset3_8_excluding_galileo_comparison/comparison.csv)

- [所有权重共同匹配点分布](../test/results/dataset3_8_excluding_galileo_comparison/common_matched_points.json)

- [2360张评估清单](../test/results/dataset3_8_excluding_galileo_comparison/test_images.txt)

- [376张Galileo排除清单](../test/results/dataset3_8_excluding_galileo_comparison/excluded_images.txt)

- [真实图来源清单](../dataset2/split_manifest.json)

- [dataset2历史四张删除记录](../dataset2/excluded_test_samples.json)

- [上传负样本数据审计](data4_negative_audit/summary.json)

- [dataset4制作验证](../dataset4/validation.json)

- [dataset5～8加载及增强验证](dataset5_8_experiments/loader_validation.json)

- [dataset3原始训练日志](../logs/train_dataset3_gpu23_b64_200epochs.log)

- [dataset5～8队列日志](../logs/dataset5_8_queue.log)

- [dataset5～8分实验训练与评估日志](../logs/dataset5_8_queue)

- [非Galileo分权重推理与关键点日志](../logs/dataset3_8_excluding_galileo)

### 12.4 对比图

![排除Galileo后六组best/last的检测与定位比较](../test/results/dataset3_8_excluding_galileo_comparison/comparison.png)

原始图表：[PNG](../test/results/dataset3_8_excluding_galileo_comparison/comparison.png)、[PDF](../test/results/dataset3_8_excluding_galileo_comparison/comparison.pdf)。

## 13. 复现实验命令

以下是实验配置复现入口。已完成训练的原名称不会自动当作新实验覆盖；如需重跑微调对照，应先选择新的run名称或队列tag。重测脚本可校验并复用已有完整结果。

### 13.1 dataset3基础训练

```bash
cd /data1/chenshiwen/landmarks_detect/landmarks2_new_em_v8.3.165
/home/ai-i-chenshiwen/miniconda3/envs/leo/bin/python -u train_gpu23_dataset3_pretrained_aug.py \
  --device 2,3 --batch-per-gpu 64 --epochs 200 --workers 24 --fresh
```

### 13.2 dataset4微调配置

```bash
/home/ai-i-chenshiwen/miniconda3/envs/leo/bin/python -u finetune_gpu01_dataset4.py \
  --data dataset4/data.yaml \
  --weights runs/earth_moon_yolov8x_pose_p6_gray_dataset3_pretrained_gpu23_b64_200epochs/weights/last.pt \
  --device 0,1 --batch-per-gpu 64 --workers 24 --epochs 20 --lr0 0.0005
```

### 13.3 dataset5～8顺序训练及评估

```bash
/home/ai-i-chenshiwen/miniconda3/envs/leo/bin/python -u run_dataset5_8_queue.py \
  --device 0,1 --batch-per-gpu 64 --workers 24 --epochs 20 --evaluate-best
```

上面是已完成这轮实验的参数入口；另起新重复实验可加`--tag 新名称`，续跑中断实验使用`--resume`。

### 13.4 全部六组排除Galileo后重测/核验

```bash
/home/ai-i-chenshiwen/miniconda3/envs/leo/bin/python -u test/compare_dataset3_8_excluding_galileo.py \
  --device 0,1
```

此处两个GPU用于分别评估不同权重，每次推理batch仍为4，不是DDP训练；脚本校验权重SHA、2360张图片集合、376张排除集合、预测及标注图数量，并生成共同匹配统计和对比报告。
