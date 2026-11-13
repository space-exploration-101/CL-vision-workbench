# 21 类目标 R 通道图像归档恢复说明

## 1. 归档身份

```text
snapshot_id: auto-landmark-Dataset-20260922-161828
source_at_backup: /data2/ai-i-chenshiwen/auto_landmark/Dataset
source_files: 136579
source_file_bytes: 897366345935
source_du_bytes: 897748054016
archives: 32
archive_bytes: 864109237775
status: VERIFIED
```

NAS 路径：

```text
Windows:
\\10.2.26.26\902_data\0-项目\13-专项\2-数据集\21类目标R通道图像归档

H200:
/mnt/ywang-nas/0-项目/13-专项/2-数据集/21类目标R通道图像归档
```

归档由 32 个相互独立的 `tar.zst` 分片组成。NAS 只保存压缩归档，不应在 NAS
目录内长期解压数据。

## 2. 恢复边界

1. 只通过 `ssh H200` 操作。
2. 恢复不需要 GPU，不启动训练、推理、量化或 FPGA 任务。
3. 不覆盖任何现有目录。恢复目标存在时立即停止。
4. 先恢复到与最终目标同一文件系统中的 `<target>.partial`，验证通过后原子改名。
5. 任一哈希、zstd、文件数、字节数或文件哈希失败时停止；保留失败现场，不发布。
6. 默认以执行恢复的用户作为文件 owner，不尝试还原原始 UID/GID。确需原 owner 时必须
   由管理员另行确认。
7. 完整恢复建议目标文件系统至少有 1 TB 可用空间。

## 3. 恢复前校验

以下命令只读 NAS：

```bash
ssh H200
set -Eeuo pipefail

NAS_ROOT='/mnt/ywang-nas/0-项目/13-专项/2-数据集/21类目标R通道图像归档'
SNAPSHOT="$NAS_ROOT/snapshots/auto-landmark-Dataset-20260922-161828"

findmnt -T "$SNAPSHOT"
df -h "$SNAPSHOT"
test -f "$SNAPSHOT/VERIFIED"
test -f "$SNAPSHOT/SHA256SUMS"
find "$NAS_ROOT" -name '*.partial' -print

cd "$SNAPSHOT"
sha256sum -c SHA256SUMS
test "$(find archives -maxdepth 1 -type f -name '*.tar.zst' | wc -l)" -eq 32
find archives -maxdepth 1 -type f -name '*.tar.zst' -print0 \
  | sort -z \
  | xargs -0 -r -n1 zstd -q -t
```

要求：

- `findmnt` 必须包含 `//10.2.26.26/902_data` 和 `cifs`；
- `.partial` 查询无输出；
- `SHA256SUMS` 和 32 个 zstd 流全部通过。

## 4. 完整流式恢复

将 `<RESTORE_TARGET>` 替换为一个尚不存在的最终目录。不要直接使用当前正在工作的
`/data2/ai-i-chenshiwen/auto_landmark/Dataset`。

```bash
set -Eeuo pipefail

SNAPSHOT='/mnt/ywang-nas/0-项目/13-专项/2-数据集/21类目标R通道图像归档/snapshots/auto-landmark-Dataset-20260922-161828'
FINAL='<RESTORE_TARGET>'
PARTIAL="${FINAL}.partial"

test ! -e "$FINAL"
test ! -e "$PARTIAL"
mkdir -p "$(dirname "$FINAL")"
mkdir "$PARTIAL"

while IFS= read -r -d '' archive; do
  echo "restoring: $archive"
  zstd -q -t "$archive"
  zstd -q -dc "$archive" \
    | tar --no-same-owner -xpf - -C "$PARTIAL"
done < <(find "$SNAPSHOT/archives" -maxdepth 1 -type f -name '*.tar.zst' -print0 | sort -z)
```

每个源文件只属于一个分片，依次解压全部 32 个分片后会重建原始目录树。

## 5. 恢复后全量验证

下面的脚本逐文件检查路径、大小和 SHA-256，并拒绝额外文件。它会读取约 897 GB，
必须等待完成。

```bash
python3 - "$SNAPSHOT/source-files.jsonl" "$PARTIAL" <<'PY'
import hashlib
import json
import os
from pathlib import Path
import sys

manifest_path = Path(sys.argv[1])
root = Path(sys.argv[2])
records = [json.loads(line) for line in manifest_path.read_text('utf-8').splitlines() if line]
expected = {item['path'] for item in records}

def sha256(path):
    digest = hashlib.sha256()
    with path.open('rb') as handle:
        while chunk := handle.read(8 * 1024 * 1024):
            digest.update(chunk)
    return digest.hexdigest()

for index, item in enumerate(records, 1):
    path = root / item['path']
    if not path.is_file():
        raise SystemExit(f'missing or non-file: {item["path"]}')
    if path.stat().st_size != item['bytes']:
        raise SystemExit(f'size mismatch: {item["path"]}')
    if sha256(path) != item['sha256']:
        raise SystemExit(f'hash mismatch: {item["path"]}')
    if index % 1000 == 0 or index == len(records):
        print(f'verified {index}/{len(records)}', flush=True)

observed = set()
for current, dirs, files in os.walk(root):
    dirs.sort()
    files.sort()
    base = Path(current)
    for name in files:
        observed.add((base / name).relative_to(root).as_posix())

if observed != expected:
    missing = sorted(expected - observed)[:10]
    extra = sorted(observed - expected)[:10]
    raise SystemExit(f'inventory mismatch; missing={missing}, extra={extra}')

total_bytes = sum(item['bytes'] for item in records)
if len(records) != 136579 or total_bytes != 897366345935:
    raise SystemExit('manifest totals do not match the frozen snapshot contract')
print(f'RESTORE_VERIFIED files={len(records)} bytes={total_bytes}')
PY
```

只有输出以下结果后才能发布：

```text
RESTORE_VERIFIED files=136579 bytes=897366345935
```

## 6. 原子发布

最终目标和 `.partial` 必须在同一文件系统：

```bash
test ! -e "$FINAL"
mv -T --no-clobber "$PARTIAL" "$FINAL"
test -d "$FINAL"
test ! -e "$PARTIAL"
printf 'snapshot_id=%s\nverified_at=%s\nfiles=%s\nbytes=%s\n' \
  'auto-landmark-Dataset-20260922-161828' "$(date --iso-8601=seconds)" \
  '136579' '897366345935' > "${FINAL}.RESTORE_VERIFIED"
```

不要通过删除或覆盖旧数据来为恢复目录让路。需要替换现有数据时，应将“恢复验证”和
“业务切换”拆成两个独立、分别授权的任务。

## 7. 只恢复单个分片

单分片适合审计或提取部分 train/val/test 数据，不代表完整数据集：

```bash
ARCHIVE="$SNAPSHOT/archives/yolo-test-0003.tar.zst"
TARGET='<EMPTY_TARGET_DIRECTORY>'
test ! -e "$TARGET"
mkdir "$TARGET"
zstd -q -t "$ARCHIVE"
zstd -q -dc "$ARCHIVE" | tar --no-same-owner -xpf - -C "$TARGET"
```

分片与文件的对应关系见 `shards.jsonl`、`shard-plan.json` 和 `filelists/*.nul`。
