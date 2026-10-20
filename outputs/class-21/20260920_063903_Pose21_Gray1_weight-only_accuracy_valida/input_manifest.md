# 任务输入清单 — Pose21 Gray1 weight-only accuracy validation

- **任务 ID**: 20260920_063903_Pose21_Gray1_weight-only_accuracy_valida
- **显示名**: Pose21 Gray1 weight-only accuracy validation
- **创建时间**: 2026-09-20T06:39:19.850056+00:00

## 模型

- **文件名**: model.pt
- **MD5**: `13b8cbcfb890d74e326659a235a71007`

## 标定集

- **来源**: 共享数据集 `pose21_gray1_50e_cal462_v1`
- **张数**: 462
- agaier_047_view_18.png
- agaier_254_view_13.png
- agaier_264_view_13.png
- agaier_288_view_33.png
- agaier_310_view_35.png
- agaier_330_view_11.png
- agaier_353_view_10.png
- agaier_364_view_29.png
- agaier_452_view_22.png
- agaier_463_view_21.png
- ... 共 462 个，完整列表见 `input_cali_files.txt`

## 测试集

- **来源**: 共享数据集 `pose21_gray1_50e_eval462_v2`（含 fpga_test_pack 子目录）
- **张数**: 462
- agaier_057_view_02.png
- agaier_057_view_08.png
- agaier_150_view_06.png
- agaier_150_view_31.png
- agaier_173_view_19.png
- agaier_173_view_25.png
- agaier_173_view_33.png
- agaier_526_view_12.png
- agaier_526_view_38.png
- agaier_526_view_39.png
- ... 共 462 个，完整列表见 `input_test_files.txt`

## 检校

- **input_validation.json**: ok=true
