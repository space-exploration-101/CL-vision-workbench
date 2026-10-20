# 21 类目标 R 通道图像归档恢复 Agent 提示词

> 使用前必须把 `<RESTORE_TARGET>` 替换为 H200 上尚不存在的最终目录。此提示词只授权
> 恢复数据到新目录，不授权覆盖、删除、训练、推理、量化、服务重启或系统修改。

```text
任务：从已验证 NAS snapshot 恢复 21 类目标 R 通道图像数据集，并在全量校验通过后
将新建的 .partial 目录原子发布为指定恢复目标。

固定输入：
- H200 连接：ssh H200
- NAS share：//10.2.26.26/902_data
- NAS mount：/mnt/ywang-nas
- NAS root：/mnt/ywang-nas/0-项目/13-专项/2-数据集/21类目标R通道图像归档
- snapshot：auto-landmark-Dataset-20260922-161828
- snapshot path：/mnt/ywang-nas/0-项目/13-专项/2-数据集/21类目标R通道图像归档/snapshots/auto-landmark-Dataset-20260922-161828
- 原备份源：/data2/ai-i-chenshiwen/auto_landmark/Dataset
- 恢复目标：<RESTORE_TARGET>
- 文件数：136579
- 文件内容字节：897366345935
- 源目录占用参考：897748054016
- 分片数：32

开始前读取并遵守：
- 项目根 AGENTS.md
- docs/agent/CONTEXT.md
- docs/agent/RUNBOOK.md
- NAS 根 00_恢复说明_README.md
- snapshot/README.md
- snapshot/snapshot.json
- snapshot/verification-report.md

强制边界：
1. 只允许 ssh H200 直连。H200 是共享服务器。
2. 本任务不需要 GPU，不检查或占用 GPU，不启动训练、推理、量化或 FPGA 任务。
3. 不停止其他用户进程，不重启服务，不修改 Docker、驱动、内核、APT、挂载配置或全局配置。
4. 不覆盖、移动或删除任何现有源数据、恢复目录或 NAS 文件。
5. 恢复目标和 <RESTORE_TARGET>.partial 任一存在时立即停止，不自行换名或清理。
6. NAS 挂载必须实时证明为 //10.2.26.26/902_data 的 CIFS；目录存在本身不能证明挂载正确。
7. 完整恢复前确认目标文件系统至少有 1 TB 可用空间，并确认 inode 足够容纳至少 136579 个文件。
8. 先完整验证 snapshot/SHA256SUMS 和全部 32 个 zstd 流。任一失败立即停止。
9. 只恢复到 <RESTORE_TARGET>.partial。所有分片成功解压且全量文件验证通过后，才允许同文件系统原子改名。
10. 默认不恢复原 UID/GID；由当前执行用户持有恢复文件。不得使用 sudo chown 冒充原 owner。
11. 失败时保留 .partial 和日志，报告精确失败点；不得自动重试覆盖、删除失败现场或发布不完整数据。
12. NAS 目录名中的“R 通道”只是业务分类，不得据此改写文件通道或执行额外预处理。

执行步骤：

Step 0：只读预检
- 记录开始时间、hostname、执行用户、恢复目标和父目录真实绝对路径。
- 使用 findmnt -T snapshot，确认实际 CIFS source 为 //10.2.26.26/902_data。
- 检查 VERIFIED、snapshot.json、SHA256SUMS、source-files.jsonl、shards.jsonl。
- 检查 NAS root 下没有 .partial。
- 检查恢复目标及其 .partial 均不存在。
- 使用 df -B1 和 df -i 检查目标父文件系统容量及 inode。
- 检查当前无另一个进程正在写入同一恢复目标。

Step 1：归档校验
- 在 snapshot 根运行 sha256sum -c SHA256SUMS。
- 对 archives/ 下按文件名排序的 32 个 *.tar.zst 逐个运行 zstd -q -t。
- 将每个归档的结果写入恢复任务日志。
- 实际归档数量不是 32、出现 .partial 或任何校验失败时停止。

Step 2：恢复到新 .partial 目录
- 在与最终目标相同的父目录创建唯一的 <RESTORE_TARGET>.partial。
- 按文件名排序，逐个执行 zstd -dc archive | tar --no-same-owner -xpf - -C partial。
- 使用 set -Eeuo pipefail，保证 zstd 或 tar 任一失败都会终止。
- 不在 NAS 上解压，不创建第二份压缩归档，不修改 snapshot。

Step 3：全量验证
- 读取 source-files.jsonl。
- 对恢复目录中的每个记录检查：相对路径存在、是普通文件、字节数一致、SHA-256 一致。
- 遍历恢复目录，确认没有 manifest 之外的额外文件。
- 最终必须得到 files=136579、bytes=897366345935。
- 再次确认最终目标仍不存在。

Step 4：发布
- 仅在所有校验通过后，将 .partial 通过同文件系统 mv -T --no-clobber 原子改名为最终目标。
- 在最终目录旁写 <RESTORE_TARGET>.RESTORE_VERIFIED，记录 snapshot_id、完成时间、文件数和字节数。
- 不删除 NAS snapshot，不删除原备份源，不执行任何业务切换。

验收条件：
- NAS snapshot SHA256SUMS 全部 PASS；
- 32/32 zstd 流测试 PASS；
- 解压过程无 zstd/tar 错误；
- 136579 个恢复文件逐文件大小和 SHA-256 PASS；
- 恢复文件总字节为 897366345935；
- 无缺失文件、额外文件和仍在发布路径内的 .partial；
- 最终目标由原子改名产生，并存在相邻 RESTORE_VERIFIED 记录；
- 原数据和 NAS 没有被修改。

最终报告格式：
task_type: raw_dataset_restore
snapshot_id: auto-landmark-Dataset-20260922-161828
status: DONE|BLOCKED|FAILED
started_at:
completed_at:
nas_mount_identity:
restore_target:
preflight:
archive_verification:
  sha256sums: PASS|FAIL
  zstd_streams: <passed>/32
restore:
  files: <count>
  bytes: <count>
  full_file_sha256: PASS|FAIL
publication:
  atomic_rename: true|false
  marker: <path or null>
changes: []
verification: []
artifacts: []
risks: []
remaining: []

具体命令以 NAS 根 00_恢复说明_README.md 为准。不要只报告计划；在所有门禁满足时完成恢复、验证和发布。
```

