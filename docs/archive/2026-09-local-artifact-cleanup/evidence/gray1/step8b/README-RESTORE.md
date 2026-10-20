# ywang YOLO Gray1 Environment Backup — Restore Guide

Snapshot: `ywang-yolo-gray1-env-20260909_124030`  
Created: 2026-09-09 CST

## Scope

Included:

- Docker image `ywang/yolo-gray1-train:ultralytics-8.3.98-v1`.
- Complete rebuild bundle for `/data3/ywang/yolo-pose-train-env`.
- Environment/package/version records.
- Training and migration documentation from `docs/train`.
- Smoke-data status and any smoke files present at snapshot time.

Explicitly excluded:

- `/data2/ai-i-chenshiwen/auto_landmark/Dataset/yolo_full` (824G, 132017 files).
- Unrelated Quantitize images, outputs, users, and system files.
- SMB credentials and all passwords.

At snapshot time the smoke-data source directory contained zero files. After Step 10 creates an approved smoke subset, back it up as a separate verified incremental artifact referencing this environment snapshot.

## Verify

From the snapshot root:

```bash
sha256sum -c manifests/SHA256SUMS
zstd -q -t runtime-images/yolo-gray1-train-ultralytics-8.3.98-v1.tar.zst
zstd -q -t build-bundles/yolo-pose-train-env-rebuild.tar.zst
find . -name '*.partial' -print
```

All hashes and stream tests must pass. No `.partial` file is a valid backup.

## Fast Image Restore

On a replacement H200 prepared with Docker Engine and NVIDIA Container Toolkit:

```bash
zstd -dc runtime-images/yolo-gray1-train-ultralytics-8.3.98-v1.tar.zst | docker load
docker image inspect ywang/yolo-gray1-train:ultralytics-8.3.98-v1
```

Do not request GPU access until the shared-server GPU readiness gate has passed.

## Project/Rebuild Restore

Restore only into a new empty ywang-owned directory:

```bash
mkdir -p /data3/ywang/yolo-pose-train-env-restored
zstd -dc build-bundles/yolo-pose-train-env-rebuild.tar.zst \
  | tar -xpf - -C /data3/ywang/yolo-pose-train-env-restored
```

Verify the restored Dockerfile, `leo-env.tar.gz`, source tree, package inventory, and hashes before rebuilding. Do not overwrite an existing project directory without separate authorization.

## Required Post-Restore Checks

- Confirm image ID/source hash against `manifests/backup-manifest.txt`.
- Run Step 7 no-network/no-GPU import smoke.
- Confirm Ultralytics imports from `/workspace/src/Yolo_fine/ultralytics` and reports `8.3.98`.
- Confirm the gray1 model contract before any training.
- Recreate smoke data only from its recorded manifest after the future incremental smoke backup exists.
