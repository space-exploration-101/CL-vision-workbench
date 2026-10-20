# 数据准备执行记录

```yaml
status: VERIFIED
executed_at: 2026-09-12T21:38:44+08:00
host: user-Rack-Server
remote_root: /data1/ywang/workflow-test/synthetic-pose21-r-only-v1
gpu_used: false
training_started: false
quantization_started: false
external_dataset_used: false
chenshiwen_data_used: false
```

## 生成内容

- 确定性合成源数据：84张8-bit、正方形、R-only PNG和84个YOLO pose标签。
- 类别：21；每个类别在train中2张、val中1张、test中1张。
- selection：`seed=20260912`、`per_class=1`，选择84项并形成train/val/test/calibration各21项。
- prepared：84张`1280×1280`单通道gray1 PNG、84个变换标签、21个cam2000 BIN。
- 总数据约179,946,841字节；raw约2,651,707字节，prepared约177,258,388字节。

## 固定实现

- grayprep repository commit：`3eae4db4eeb82425d1230d44e1feacbed6054f47`。
- 依赖镜像：`ywang/yolo-gray1-data-pipeline:0.1.1`。
- image ID：`sha256:2f172eb582ba0eb7cb6192cf0449893baef3b0bb88dfb8cc83b9811839b3e3d3`。
- 该镜像中的CLI早于并发实现，因此执行时只读挂载上述commit的`src/configs/schemas`，镜像仅提供固定Python/OpenCV/NumPy依赖。
- 并发：32 workers × 1 OpenCV thread。

## 验证结果

```yaml
grayprep_status: PASS
items_checked: 84
hashes_checked: 195
bins_checked: 21
class_count: 21
split_class_coverage: PASS
cross_split_stem_overlap: none
cross_split_content_overlap: none
partial_files: 0
```

关键hash见`DATASET_INVENTORY.json`。

## 被拒绝的首次候选

首次合成器没有把split编码进图像内容，导致val/test出现相同内容。grayprep严格验证以`content_overlap:val:test`拒绝了候选，没有发布最终目录。随后：

1. 修正生成器，使split参与确定性像素变体。
2. 验证暂存目录解析为`/data1/ywang/workflow-test/.synthetic-pose21-r-only-v1.partial`且最终目录不存在。
3. 删除仅属于本任务的未发布暂存目录并完整重建。
4. 第二次候选通过全部门禁。

失败记录保留在本文，避免未来重复同类生成错误。

## 最终发布

```yaml
published_at: 2026-09-12
published_path: /data1/ywang/workflow-test/synthetic-pose21-r-only-v1
published_bytes: 179949150
inventory_sha256: be64a0d7a784c8bdbd9ce617b0da1abc276cc371df88128d16db33e69d734ba9
verified_marker_sha256: 725e28e2cc55eb06378de03273d7d94ff487c9a12a8fc9d7f35e67526d164569
raw_sha256sums: PASS
prepared_sha256sums: PASS
post_publish_grayprep_verify: PASS
partial_files: 0
```

候选目录通过校验后由同一文件系统内的 `.partial` 路径发布为最终路径。发布后再次运行 `grayprep dataset verify-smoke`，结果为84项、195个hash、21个BIN、21类，且 `failures=[]`。
