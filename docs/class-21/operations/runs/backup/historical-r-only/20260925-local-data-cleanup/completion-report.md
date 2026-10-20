# 旧 21 类 R-only 本地数据归档与清理完成报告

> 执行日期：2026-09-25  
> 状态：`VERIFIED`  
> 本地清理：完成

## 1. 结论

原 `E:\codex\quantitize\data` 是 2026-09-03/04 的 21 类三通道 R-only
过渡实验副本，不属于当前单通道 Gray1 基线。旧权重和重建资料已经归档到：

```text
\\10.2.26.26\902_data\0-项目\13-专项\2-数据集\21类目标R通道图像归档\历史实验\20260903-04_21类R-only过渡实验
```

NAS 归档完成独立校验后，本地 `data/` 已整体删除。

## 2. 本地资产身份

| 项目 | 数量或身份 |
|---|---:|
| 标定图片 | 800 张 |
| 测试图片 | 84 张 |
| 测试标签 | 84 份，其中 17 份为空标签 |
| 标定与测试重复图片 | 9 张，SHA-256 相同 |
| 唯一原始文件 | 959 个 |
| `best.pt` | 199,053,642 bytes |
| `best.pt` SHA-256 | `94125be741cc05bb2df3f6d17d15fb07b182fb74011c2f5ab2669be1c8f89e2f` |

权重内部元数据指向旧训练运行
`landmarks_yolov8x_p6_pose_pretrained_1024_no_online_aug_b64_run2`、
`yolo_full/dataset.yaml` 和 `imgsz=1024`。它不是当前 Gray1 权重。

## 3. 可恢复性验证

- 原始快照：`auto-landmark-Dataset-20260922-161828`，状态 `VERIFIED`。
- 968 条本地角色记录逐个计算 SHA-256，全部与快照 `source-files.jsonl` 一致。
- 图片和标签不在历史实验目录重复保存；通过
  `metadata/selection-manifest.jsonl` 从原始快照恢复。
- 旧权重、历史任务配置、选择清单、源快照身份和校验摘要均已归档。
- NAS 归档共 10 个文件、199,461,606 bytes；`SHA256SUMS`、权重 ZIP 结构和
  JSON 元数据解析均通过，目录内无 `.partial`。

发布后的独立校验结果：8 个清单内文件哈希全部通过、968 条选择记录全部为
`source_match=true`、权重哈希一致。

首次发布后的独立校验发现 `archive-manifest.json` 末尾存在字面量换行标记。
在删除本地数据前，归档被退回 `.partial-repair`、移除 `VERIFIED`、修正并重新
计算全部哈希；最终再次独立校验通过后才重新发布。

## 4. 本地清理结果

```text
deleted_path: E:\codex\quantitize\data
deleted_files: 971
deleted_bytes: 12067205064
deleted_gib: 11.238
exists_after: false
```

清理前确认目标解析后严格等于工作区直属 `data`，目录内不存在重解析点。
本次未使用 GPU，未访问 H200，未启动训练、推理、量化或 FPGA 任务，也未重启服务。

