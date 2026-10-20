# auto_landmark Dataset NAS 归档完成报告

## 结论

`/data2/ai-i-chenshiwen/auto_landmark/Dataset` 已按独立 `tar.zst` 分片完整归档到
NAS，并通过发布时校验、train/val/test 恢复抽样和发布后的独立全量校验。

```text
status: VERIFIED
snapshot_id: auto-landmark-Dataset-20260922-161828
source: /data2/ai-i-chenshiwen/auto_landmark/Dataset
nas_share: \\10.2.26.26\902_data
nas_snapshot: \\10.2.26.26\902_data\0-项目\13-专项\2-数据集\21类目标R通道图像归档\snapshots\auto-landmark-Dataset-20260922-161828
h200_snapshot: /mnt/ywang-nas/0-项目/13-专项/2-数据集/21类目标R通道图像归档/snapshots/auto-landmark-Dataset-20260922-161828
```

## 归档规模

| 项目 | 数值 |
|---|---:|
| 源文件数 | 136,579 |
| 源文件内容字节 | 897,366,345,935 |
| 源目录占用字节（预检） | 897,748,054,016 |
| 归档分片数 | 32 |
| 归档总字节 | 864,109,237,775 |
| 归档/源内容字节比 | 96.2939% |
| 节省字节 | 33,257,108,160 |
| 节省比例 | 3.7061% |

PNG 数据本身已压缩，因此 `zstd -3` 的额外压缩收益有限。归档保留压缩分片，NAS
不长期保存解压副本。

## 分片和发布

- 目标分片源大小约 32 GiB。
- `yolo_full` 的图片与同 stem 标签成对进入同一 train/val/test 分片。
- 每个分片先写为 `.partial`，再依次执行 `zstd -t`、tar 成员数检查和 SHA-256，
  通过后原子改名。
- 最终包括 21 个 train、3 个 val、3 个 test 分片，以及 `aituosha`、`full`、
  `moon`、`onboard`、`metadata` 5 个附属分片。
- 最终 NAS 根和 snapshot 内均无 `.partial`。

备份开始于 `2026-09-22 16:25:58 CST`，snapshot 于
`2026-09-23 00:19:07 CST` 发布，耗时约 7 小时 53 分钟。

## 发布时校验

- 源 path/size/mtime 清单在归档前后保持不变。
- 32/32 个归档通过 zstd 流测试、tar 成员计数和 SHA-256。
- train 恢复抽样：`yolo-train-0021`，3,144 个文件，PASS。
- val 恢复抽样：`yolo-val-0003`，2,986 个文件，PASS。
- test 恢复抽样：`yolo-test-0003`，2,792 个文件，PASS。
- 恢复后的抽样文件逐个与源 manifest 中的大小和 SHA-256 比对。

NAS 权威记录：

```text
snapshot.json
source-files.jsonl
shard-plan.json
shards.jsonl
SHA256SUMS
verification-report.md
VERIFIED
catalog/snapshots.jsonl
```

## 独立校验

发布完成后使用独立脚本重新只读检查，结果如下：

- NAS 身份为 `//10.2.26.26/902_data`、文件系统为 CIFS。
- 完整 `SHA256SUMS` 校验 PASS。
- 32/32 个归档重新执行 `zstd -t`，全部 PASS。
- snapshot、shard manifest、实际文件和 catalog 的数量/字节一致。
- 源目录当前 path/size/mtime 清单仍与 `source-files.jsonl` 一致。
- 独立校验于 `2026-09-23 05:45:17 CST` 完成。

校验工具：

```text
docs/class-21/operations/tools/verify_dataset_snapshot.py
SHA-256: b9c59ecf1baf4b875e1bd965655988a6e7f8b37fe47a62a4588f132b6d17847b
```

H200 证据日志：

```text
/data3/ywang/backup-jobs/auto-landmark-Dataset-20260922-161828/run.log
/data3/ywang/backup-jobs/auto-landmark-Dataset-20260922-161828/independent-verification.log
```

## 恢复约束

恢复前必须先验证 `SHA256SUMS` 和所需分片的 `zstd -t`。解压目标必须是新的
`.partial` 目录；完成文件数、字节数和哈希验证后再原子发布，不得覆盖训练中的数据集。

NAS 目录名称中的“R 通道”是业务分类，不能代替文件级输入语义。实际数据语义应以
数据合同和 manifest 为准。

## NAS 恢复文档

`2026-09-23` 在 NAS 归档根新增以下恢复入口，未修改已验证 snapshot：

```text
00_恢复说明_README.md
SHA-256: ee85ce4550ff148f12864dd7d38e07f75d13fbb7e68561bfdf86bedcd0366dcf

01_恢复_AGENT提示词.md
SHA-256: eea924613f183b6e208c967715f0bf8d1e1197322c6567ab0f63cc490c35dfef

RESTORE_DOCS_SHA256SUMS
```

恢复说明覆盖归档校验、32 分片流式解压、逐文件全量哈希验证、原子发布和单分片
恢复。Agent 提示词要求由用户先填写一个尚不存在的 `<RESTORE_TARGET>`，不授权覆盖、
删除、GPU 任务、服务重启或业务切换。

## 未执行操作

- 未删除、移动或修改源数据。
- 未在 NAS 上保留解压副本。
- 未使用 GPU，未启动训练、推理、量化或 FPGA 任务。
- 未重启服务，未停止其他用户进程，未修改 Docker 或系统配置。
