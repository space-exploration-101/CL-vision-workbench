# 早期 cam2000 PNG/BIN 转换原型

> 归档日期：2026-09-18  
> 原始位置：`docs/data-preprocess/`  
> 状态：历史协议样本，不是当前数据预处理入口

## 归档内容

```text
legacy_png_cam2000_bin_converter.py
cam2000_u12_2000x2000_legacy_sample.bin
```

`legacy_png_cam2000_bin_converter.py` 是独立数据预处理仓库建立前使用的 PNG/BIN 转换原型，支持：

- PNG 灰度化并缩放到 `2000x2000`；
- uint8 按 `round(v * 4095 / 255)` 映射为 12 bit；
- 两个 12-bit 像素打包为三个字节；
- 每行 3,000 字节有效载荷、stride 4,096 字节、行尾补零；
- BIN 反向解码为 PNG 和往返误差统计。

`cam2000_u12_2000x2000_legacy_sample.bin` 原文件名为缺少语义的 `2_1.bin`。归档前只读检查确认：

```text
文件大小：8,192,000 bytes
图像尺寸：2000x2000
行数：2000
stride：4096 bytes
有效载荷：3000 bytes/row
padding：1096 bytes/row
非零 padding 行数：0
```

该样本缺少源图片、生成命令和 manifest，因此只能证明文件外形符合协议，不能作为权威 golden 向量或精度基线。

## 当前替代实现

当前正式实现位于：

```text
H200：/data3/ywang/yolo-gray1-data-pipeline
版本：0.2.1
协议：docs/CAMERA_BIN_ABI.md
配置：configs/cam2000.yaml
```

原型脚本默认对三通道图片执行 `BGR2GRAY`，缺少当前实现中的 `native_gray` / `r_only` 显式输入语义门禁。不得直接用于当前 Gray1 数据生成、量化标定或 FPGA 交付。
