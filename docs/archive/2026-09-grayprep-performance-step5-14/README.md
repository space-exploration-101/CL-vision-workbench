# Grayprep Step 5/14 性能实验归档

> 归档日期：2026-09-18  
> 原始位置：`_step5_upload/`  
> 状态：历史性能实验与上传暂存脚本，不是当前执行入口

## 1. 归档内容

`raw/` 原样保存 16 个文件、73,024 字节，分为三类：

1. Step 5 OpenCV 线程扫描的运行、等待、分析和汇总脚本；
2. Step 14 GPU PoC 的探针、启动和等待脚本；
3. 2026-09-11 的性能实验记录与当时的汇总报告。

主要文件：

```text
raw/run_step5.py
raw/20260911-104349_opencv_thread_scan_lean_n840.md
raw/PERFORMANCE_REPORT.md
raw/run_step14.py
raw/gpu_poc.py
raw/pipeline_thread.py
```

## 2. Step 5 的有效结论

历史实验使用数据处理镜像 `ywang/yolo-gray1-data-pipeline:0.1.1`、固定 840-item manifest 和 H200 共享负载环境，比较不同 `cv2.setNumThreads()` 设置。

当时结论：

- `T=8/16/32/128` 的端到端吞吐差异小于 3%；
- 按资源较少优先的规则选择 `opencv_threads=8`；
- net1280、cam2000 和 combined 相对基线变化约为 `+0.07%`、`+0.97%`、`+1.09%`；
- PNG `imdecode` 是主要源读取热点，增加 OpenCV 线程不是主要加速手段；
- cam2000 pack/decode 的 Python 行循环和图片级串行处理是后续优化方向。

该实验结果属于当时版本和负载条件。当前性能结论应以 H200 数据处理仓库的最新 `docs/perf_test/PERFORMANCE_REPORT.md` 为准。

## 3. Step 14 脚本边界

`run_step14.py`、`gpu_poc.py`、`probe_gpu.py`、`start_step14.sh` 和 `wait_step14.py` 是后来加入同一上传暂存目录的 GPU PoC 工具。它们写死了历史实验路径、GPU UUID和旧镜像版本，只用于还原当时测试方法。

这些脚本没有作为正式 `grayprep` CLI 发布，不能直接用于当前主机或当前数据流水线。需要复现实验时，应从当前仓库重新实现最小 benchmark，并重新执行 GPU 资源门禁。

## 4. 当前替代位置

数据处理正式仓库：

```text
H200：/data3/ywang/yolo-gray1-data-pipeline
检查时 HEAD：f147cee93c83876348205ca268eb930010192116
```

本地 `.work/yolo-gray1-data-pipeline` 已逐文件与 H200 当前仓库核对，所有本地改动均已有远端等价内容，因此未随本目录归档。

当前执行依据：

- H200 仓库 `README.md`
- H200 仓库 `docs/perf_test/PERFORMANCE_REPORT.md`
- `../../class-21/knowledge/02_21类目标数据与输入预处理.md`
- `../../reference/PERFORMANCE.md`
- `../../agent/RUNBOOK.md`

不得把 `raw/PERFORMANCE_REPORT.md` 的 `0.1.1` 状态覆盖到当前报告，也不得把上传脚本重新放入 `/tmp` 后直接执行。
