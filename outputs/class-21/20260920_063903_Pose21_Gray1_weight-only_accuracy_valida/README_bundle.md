# 量化产物包（精简）

见 WEB_DESIGN.md §6.2。
不含 input/model.pt、共享标定/测试集原始图、fpga_test_pack/。

成果 workspace ONNX 仅含 `pose21_gray1_weight_only_accuracy_output.onnx`（主推理模型）。
不含：`pose21_gray1_weight_only_accuracy.onnx`、`*_fp16*.onnx`（量化中间文件，仍在任务 workspace/ 内）。

含 `workspace/renamed_weights_bin/` 每层 01_rename wt + 03_rename bn，及 `workspace/all_bin/` 合并权重。
不含 `workspace/bin/` 原始层调试 bin。
