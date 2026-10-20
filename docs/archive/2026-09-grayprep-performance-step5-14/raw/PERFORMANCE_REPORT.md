# Grayprep Performance Report

这是持续更新的性能汇总。只记录通过正确性门禁、条件可复现的实验结果；完整证据放在 `experiments/<experiment_id>.md`。

## 1. Current status

```yaml
report_updated_at: 2026-09-11
pipeline_version: 0.1.1
latest_valid_experiment: experiments/20260911-104349_opencv_thread_scan_lean_n840.md
baseline_status: MEASURED_SHARED_LOAD
completed_steps: [0, 1, 2, 3, 4, 5]
current_step: 6
current_step_status: PLANNED
recommended_net1280_workers: 1  # baseline only; not optimized
recommended_cam2000_workers: 1  # baseline only; not optimized
recommended_opencv_threads: 8
gpu_recommendation: NOT_EVALUATED
```

## 2. Confirmed implementation facts

- 数据集处理在 `build_smoke_dataset()` 中逐 item 串行执行。
- 每个 item 先执行 `net1280`；只有 test split 执行 `cam2000`。
- 应用级图片并发为 1，没有 worker/batch 参数。
- 当前实现为 CPU OpenCV/NumPy；容器不申请 GPU。
- `cam2000` 的 encode/decode 各包含 2000 行 Python 循环，并对每个输出执行完整内存回环。
- `uint8→uint12` 当前使用 float64 中间数组。
- PNG 编码、文件 fsync、重复哈希和 BIN 写入可能形成存储或 CPU 瓶颈。
- `cv2.setNumThreads(T)` 在 T=8/16/32/128 下端到端吞吐差异小于 3%；PNG `imdecode` 约占源读取+解码时间的 99%，且基本不随 T 扩展。

## 3. Baseline

基线使用固定 840-item manifest、单进程、OpenCV 默认 128 线程。源数据位于 `/data2`，输出位于 `/data1`。主机有其他用户负载，但三轮结果波动均小于 1.3%。

| pipeline | dataset | workers | OpenCV threads | median items/s | P95 ms | peak RSS MiB | output GiB | experiment |
|---|---|---:|---:|---:|---:|---:|---:|---|
| net1280 | p10: 840 | 1 | 128 | 2.2664 | 553.754 | 275.2 | 0.447 | [baseline](experiments/20260910-234757_baseline_cpu_single_process_n840.md) |
| cam2000 | p10 test: 210 | 1 | 128 | 1.9663 | 618.276 | 264.4 | 1.602 | [baseline](experiments/20260910-234757_baseline_cpu_single_process_n840.md) |
| combined | p10: 840 net + 210 bin | 1 | 128 | 2.1114 | not instrumented | 326.3 | 2.050 | [baseline](experiments/20260910-234757_baseline_cpu_single_process_n840.md) |

Effective average CPU use was only about 1.14–1.17 cores despite `opencv_threads=128`. The configured thread count is therefore not evidence of full CPU utilization.

## 4. Valid experiments

| date | change | scope | baseline items/s | candidate items/s | delta | correctness | decision | record |
|---|---|---|---:|---:|---:|---|---|---|
| 2026-09-10/11 | establish baseline | CPU single-process p10 | n/a | net 2.2664; cam 1.9663; combined 2.1114 | n/a | PASS | ACCEPT | [record](experiments/20260910-234757_baseline_cpu_single_process_n840.md) |
| 2026-09-11 | cv2.setNumThreads scan | CPU single-process p10 | net 2.2664; cam 1.9663; combined 2.1114 | net T=8 2.2680; cam T=8 1.9853; combined T=8 2.1344 | net +0.07%; cam +0.97%; combined +1.09% | PASS | ACCEPT T=8 | [record](experiments/20260911-104349_opencv_thread_scan_lean_n840.md) |

## 5. Rejected or invalid experiments

| date | change | reason | record |
|---|---|---|---|

失败记录不得删除。它们用于避免后续 Agent 重复无效方向。

## 6. Current recommendation

单进程最佳 OpenCV 线程数为 **8**：与 16/32/128 的中位吞吐差异小于 3%，按规则取更少线程。T=1 大约慢 4%，没有必要作为默认。

阶段计时与拆分测试显示真正热点是：

1. 源 PNG `imdecode`：平均约 202 ms，P50 约 305 ms，占 open/read+decode 的 99%；
2. `cam2000` pack：约 66.4 ms/case；
3. round-trip decode：约 44.2 ms/case；
4. uint8→uint12：约 28.5 ms/case；
5. PNG encode：约 27.4 ms/case。

`open/read` 只有约 2 ms，iowait ≈ 0%，所以当前不是磁盘瓶颈。后续必须同时报告 compute-only 与 end-to-end。

优先级：

1. 向量化 `cam2000` 全帧 pack/decode。
2. 优化 uint8→uint12 和内存复制。
3. 在单图路径稳定后增加图片级多进程，以并行化 PNG 解码。
4. 仅在计算阶段确实主导时评估 GPU。

## 7. Next experiment

```yaml
step: 6
name: vectorize_cam2000_pack_decode
data: FPGA golden, then 210-frame p10 cam2000, correctness vs Step 4/5 BIN hashes
changed_variable: replace 2000-iteration Python pack/decode loops with whole-frame reshape/slice
gate: FPGA golden SHA-256 unchanged; all BIN bytes identical
accept: cam2000 median throughput +10% with acceptable RSS and tail latency
opencv_threads: 8
gpu: not used
status: PLANNED
```

进度：Step 0–5 已完成。Step 5 选择 `recommended_opencv_threads=8`，确认线程扫描不是端到端加速手段。

## 8. Update rules

每个 `VALID`、`INVALID` 或 `SUPERSEDED` 实验结束后：

1. 在对应表格增加一行并链接实验记录。
2. 只用三轮正式结果的中位数比较。
3. 明确数据量、worker、OpenCV threads 和存储路径。
4. 若推荐值改变，更新 Current status 和 Current recommendation。
5. Next experiment 始终只保留一个最小、可证伪的下一步。
6. 不把大日志、图片、BIN、数据集、模型或凭据提交到 Git。
