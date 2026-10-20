# Performance Experiment `20260911-104349_opencv_thread_scan_lean_n840`

```yaml
experiment_id: 20260911-104349_opencv_thread_scan_lean_n840
status: VALID
started_at: 2026-09-11T10:46:53+08:00
finished_at: 2026-09-11T13:04:45+08:00
operator: Codex agent
objective: execute OPTIMIZATION_PLAN Step 5 OpenCV thread scan on frozen p10
hypothesis: cv2.setNumThreads(T) changes single-process throughput; fewer threads may match or beat the OpenCV default of 128
single_changed_variable: cv2.setNumThreads(T)
authorization: user explicitly requested running OPTIMIZATION_PLAN.md tests
```

## 1. Versions

```yaml
host: user-Rack-Server
git_commit: 62e27bb47bb17e5ef9c4c465b724932633c7baf5
git_status: untracked docs/perf_test only; pipeline implementation unchanged
docker_image: ywang/yolo-gray1-data-pipeline:0.1.1
docker_image_id: sha256:2f172eb582ba0eb7cb6192cf0449893baef3b0bb88dfb8cc83b9811839b3e3d3
image_revision: 5787aa904ff80a71d585a527bc4ad973c288e48a
python: 3.12.13
numpy: 1.26.4
opencv: 4.11.0
opencv_parallel_framework: pthreads
opencv_opencl: false
net1280_config_sha256: c71395417c3e3fb6d86abe89ae52ba0336c8767bbd4f6a99551e04cec4588f33
cam2000_config_sha256: a79b1aee1a72c7e418f7ec7abb2881adf9c4e4b80bfd549c6a188a7fd97da757
```

The image revision is the v0.1.1 code. The repository HEAD adds restore documentation after v0.1.1 and does not change the pipeline implementation. Step 5 did not modify pipeline source.

## 2. Resource precheck

```yaml
logical_cpus: 128
cpu_affinity: 128
workers: 1
memory: approximately 1 TiB host memory
input_filesystem: /data2, ext4 on /dev/sdc1, 341 GiB free at start
output_filesystem: /data1, ext4 on /dev/nvme0n1p1, 440 GiB free at start
gpu_requested: false
gpu_used_by_experiment: false
initial_host_load_average: 27.38, 27.11, 26.26
```

The H200 host was shared. Another user had an active GPU-3 training job and many CPU Python workers. Those processes were not changed or stopped. Quantitize API and Web remained healthy. The experiment used the same frozen p10 manifest, image, source path, and output disk as Step 4.

## 3. Dataset

```yaml
dataset_id: yolo_full
source_path: /data2/ai-i-chenshiwen/auto_landmark/Dataset/yolo_full
source_read_only: true
source_semantics: r_only
selection_p10_sha256: 39821029b39425f1cdddd823e6c68a0e8eb9011c23dd9c6da565cf476ab3852b
p10_items: 840 net1280, 210 cam2000
```

## 4. Compared configurations

### Baseline (Step 4)

```yaml
commit: 62e27bb47bb17e5ef9c4c465b724932633c7baf5
workers: 1
opencv_threads: 128
implementation: current pipeline, no code change
storage_path: /data1/ywang/grayprep-perf/20260910-234757_step0-4
```

### Candidate

```yaml
commit: 62e27bb47bb17e5ef9c4c465b724932633c7baf5
workers: 1
opencv_threads: scanned T in {1,4,16,128} then {8,16,32}
implementation: same pipeline; only cv2.setNumThreads(T) in the out-of-tree harness
storage_path: /data1/ywang/grayprep-perf/20260911-104349_opencv_thread_scan_lean_n840
```

Warm-up ran once at T=128 and was not scored. Phase A used one timed run per pipeline and T. Phase B used three timed runs for T={8,16,32}. Combined ran once at the selected T=8.

## 5. Correctness gate

```yaml
unit_tests: not re-run; pipeline bits unchanged from Step 2/4
t1_net1280_vs_step4: PASS (1680 png/txt files identical)
t1_cam2000_vs_step4: PASS (210 bin files identical)
t8_net1280_vs_step4: PASS (1680 png/txt files identical)
t8_cam2000_vs_step4: PASS (210 bin files identical)
combined_sha256sums_vs_step4: PASS (byte-identical SHA256SUMS)
combined_verify_smoke: PASS
partial_files_found: 0
errors: 0
```

Raising OpenCV threads did not change pixels, labels, or BIN bytes.

## 6. Measurements

Phase A single runs (not used for the median decision):

| T | net1280 items/s | cam2000 items/s | geometric mean |
|---:|---:|---:|---:|
| 1 | 2.1767 | 1.9836 | 2.0779 |
| 4 | 2.2446 | 1.9799 | 2.1081 |
| 16 | 2.2736 | 2.0099 | 2.1377 |
| 128 | 2.2705 | 1.9736 | 2.1168 |

Phase B three-run medians:

| config | run | wall_s | cpu_s | items | items/s | p50_ms | p95_ms | peak_rss_mib | write_mib_s | io_wait_pct |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| net1280 T=8 | 1 | 370.768 | 405.095 | 840 | 2.2656 | 454.098 | 554.450 | 272.9 | 1.46 | 0.001 |
| net1280 T=8 | 2 | 370.119 | 403.835 | 840 | 2.2695 | 454.177 | 553.291 | 273.3 | 1.46 | 0.001 |
| net1280 T=8 | 3 | 370.368 | 403.848 | 840 | 2.2680 | 454.599 | 553.309 | 273.6 | 1.46 | 0.001 |
| cam2000 T=8 | 1 | 105.971 | 108.728 | 210 | 1.9817 | 505.391 | 616.031 | 264.8 | 15.23 | 0.003 |
| cam2000 T=8 | 2 | 105.777 | 108.470 | 210 | 1.9853 | 503.993 | 612.801 | 265.0 | 15.30 | 0.003 |
| cam2000 T=8 | 3 | 104.077 | 106.891 | 210 | 2.0177 | 496.305 | 608.903 | 263.7 | 15.75 | 0.005 |
| net1280 T=16 | median | | | 840 | 2.2643 | | | | | |
| cam2000 T=16 | median | | | 210 | 1.9867 | | | | | |
| net1280 T=32 | median | | | 840 | 2.2726 | | | | | |
| cam2000 T=32 | median | | | 210 | 1.9798 | | | | | |
| combined T=8 | 1 | 393.551 | 431.753 | 840 | 2.1344 | n/a | n/a | 316.2 | 5.57 | 0.002 |

Phase B variation: net1280 ≤ 0.46%, cam2000 ≤ 2.30%. All within the 5% gate.

Effective CPU (cpu/wall) for net1280: T=1 ≈ 0.999 cores; T=8 ≈ 1.09; T=16/128 ≈ 1.14. Disk was idle: iowait ≈ 0%, `/data1` util < 0.4%, `/data2` read ≈ 0.3–0.5 MiB/s.

Split test (80 images: 40×4096 and 40×2048, T=1, once):

| stage | mean_ms | p50_ms | p95_ms | share |
|---|---:|---:|---:|---:|
| open/read | 1.970 | 1.509 | 5.970 | 0.97% |
| cv2.imdecode | 201.625 | 304.530 | 431.405 | 99.03% |
| cv2.imread combined | 203.538 | 308.365 | 435.108 | n/a |

## 7. Result

```yaml
baseline_median_items_s: net 2.2664; cam 1.9663; combined 2.1114
candidate_median_items_s: net T=8 2.2680; cam T=8 1.9853; combined T=8 2.1344
throughput_delta_pct: net +0.07; cam +0.97; combined +1.09
baseline_variation_pct: net 0.53; cam 1.21; combined 0.27
candidate_variation_pct: net 0.18; cam 1.82
correctness_passed: true
resource_regression: false
decision: ACCEPT
selected_opencv_threads: 8
```

T={8,16,32} geometric means differ by less than 3%, so the selection rule prefers the fewest threads: **T=8**. T=16 was the Phase A single-run winner, but the three-run medians do not support keeping 16 or 32 threads.

OpenCV thread count is not the end-to-end bottleneck. PNG `imdecode` dominates source ingest (~200 ms mean, ~305 ms P50) and is effectively single-threaded. The application still processes one image at a time, so extra OpenCV threads only affect short kernels such as resize. cam2000 pack/decode remain Python row loops and ignore `cv2.setNumThreads`. Disk wait is negligible.

## 8. Artifacts and cleanup

```yaml
small_artifacts_committed:
  - docs/perf_test/experiments/20260911-104349_opencv_thread_scan_lean_n840.md
large_artifacts_not_committed:
  - /data1/ywang/grayprep-perf/20260911-104349_opencv_thread_scan_lean_n840
output_path: /data1/ywang/grayprep-perf/20260911-104349_opencv_thread_scan_lean_n840
cleanup_status: non-kept timed outputs deleted after metrics; T=1, T=8 correctness, and combined T=8 outputs kept
recoverability: frozen p10 manifest and Step 4 baseline outputs remain on /data1
```

## 9. Next step

Step 6: vectorize cam2000 full-frame pack/decode to remove the 2000-iteration Python row loops, with FPGA golden and byte-identical BIN gates. Keep `cv2.setNumThreads(8)` unless a later experiment revisits it.
