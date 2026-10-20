# Pose21 Gray1 NAS 备份与环境恢复

> 文档整理日期：2026-10-18；环境备份证据截至 2026-10-16，本机 PT 临时副本证据截至 2026-10-20
> 当前顶层状态：`PARTIAL`
> 当前环境备份基线：`20261016_H200_docker_images`，四类镜像及受跟踪源码为 `ASSET_VERIFIED`；数据与结果沿用各自独立归档。

> 位置复核：2026-10-15 已只读核对 NAS 目录与小型清单；Gray1 备份根现为 `20260922_21目标灰度量化数据`，原 `20260922_21目标灰度` 路径不可用。下文已更新当前位置，历史哈希仍沿用原验收记录，本轮未重算大文件。

> 10.16 版于 14:03 完成统一归档：训练目录 22 项文件、预处理/API/Web 各 9 项文件全部通过校验。训练原备份曾完成现有 H200 daemon 导入与 CPU 复测；新增三个归档完成配置/层哈希和 NAS 回读验证，未执行导入。统一目录见第 4 节，镜像恢复入口见第 6.4 节。

> 除第 3.6 节的本机 PT 副本外，本文沿用已有备份记录，未重新验收 H200、旧数据集与结果。

NAS 是网络存储；镜像保存软件环境，源码保存程序，模型与数据需要分别恢复。SHA-256 是内容指纹，隔离恢复是在不依赖现用系统的独立环境验证能否重新运行。

10.18 最新 main 与 10.16 快照身份分别记录，见[源码身份](../agent/topics/REPOSITORIES.md)。本文件中的“当前镜像”指最近已验证的恢复基线，不代表新 main 已部署。

本文是平台备份覆盖、归档资产身份和恢复缺口的唯一详细维护来源；其中 Pose21 Gray1 的资产仍绑定原模型与数据。第 2–4 节记录备份时点的身份，不随最新 main 或模型更新自动变化；当前模型身份见[MODEL_BASELINE](../agent/topics/MODEL_BASELINE.md)，最新源码见[REPOSITORIES](../agent/topics/REPOSITORIES.md)。训练、测试、标定和量化的完整实验背景见[完整报告](../class-21/reports/current/20260920_Pose21_Gray1_训练测试标定与量化完整报告.md)。

## 1. 状态与结论

| 状态 | 含义 |
|---|---|
| `PARTIAL` | 只保护部分资产，不能完整重建训练或量化环境 |
| `ASSET_VERIFIED` | 单项资产已完成复制、格式/内容和身份校验 |
| `COMPLETE` | 计划资产齐全，但尚未做隔离恢复 |
| `RECOVERY_VERIFIED` | 已在隔离环境完成恢复和验收 |

当前已验证的旧版 21 类目标数据集、训练结果、6,600 张 PT 推理结果和最终 weight-only 量化结果均已有 NAS 副本，并通过逐文件比较或 SHA-256 校验，因此这些单项资产是 `ASSET_VERIFIED`。包含旧源 `yolo_full` 的整个 `auto_landmark/Dataset` 也已形成独立 `VERIFIED` 快照。

顶层状态仍是 `PARTIAL`。训练/PT、预处理、量化 API 和 Web 四个当前镜像及对应受跟踪源码已归档；独立离线构建材料、部署凭据与其他平台资产仍未形成完整恢复闭环，也没有执行 clean-room 恢复、6,600 张量化 ONNX 全量回归和真实 FPGA 闭环。不能把单项资产校验等同于整套系统可恢复。

## 2. 必须保护的身份

- 数据代码 commit：`f147cee93c83876348205ca268eb930010192116`。
- 历史训练基线 commit：`dd70691faa8eb276301717d3d182b624cb653d14`；Predictor 修复已提交于 `5b3caca445874b3567c0ad0b425c02c8d6c920e2`。日期镜像以该提交构建；打包说明和默认入口发布提交为 `c981f5b8a7a4f3c8a111c63812d0c74f5fe8b5f2`，为当日发布快照，不要求该临时分支名仍存在。
- 当前训练/PT 镜像：`yolo-gray1:1280-train-predict-20261016`，完整 ID `sha256:79b0086e39b83c8a3bbb76bfe2df1d8683bd443a6bef19ee7d2e69f096033635`。
- 量化平台历史开发基线：`c45eb80abbe582a9194cb310a2be66dc5cfbabab`。
- 当前已验证的旧版 21 类目标数据集：`pose21-gray1-1280-m-v1`，包含 train/val/test。
- 正式模型：`best.pt`，SHA-256 `622cff54db472b61c85db03648b36c119f030eacbd4cd87cf5d2dda67e69d6a7`。
- 正式 PT test 运行：`pose21_gray1_m_50e_test_eval_20260918_212142`，状态 `PASS`。
- 最终量化任务：`20260920_063903_Pose21_Gray1_weight-only_accuracy_valida`。
- 最终量化 bundle SHA-256：`4d858c72cf5a81509fe2fa10e500cce8aefb7c7ccd69094859d87e98908fde22`。

镜像归档必须记录不可变 image ID；`latest` 只能作为标签，不能作为身份。

10.16 环境备份的四个镜像身份如下，恢复后应逐一比对完整 ID：

| 用途 | 镜像标签 | 完整 image ID |
|---|---|---|
| 训练/PT 推理 | `yolo-gray1:1280-train-predict-20261016` | `sha256:79b0086e39b83c8a3bbb76bfe2df1d8683bd443a6bef19ee7d2e69f096033635` |
| 预处理 | `ywang/yolo-gray1-data-pipeline:0.2.1` | `sha256:ae238a8a9099ef31a6c06ef788d964ec9dcc4875f80958cd765c1614b90892db` |
| 量化/API | `quantitize-platform-api:latest` | `sha256:f633b01c2f85d1fd39bf01e9b9f556056d03c805529781c8879e3172412e8ed2` |
| Web | `quantitize-platform-web:latest` | `sha256:7aef48747d3fc4270f85133382ffbcf83ea0c53e831f06978ac777b48743d9e9` |

平台宿主源码快照为 `b1ac5ea84e0374d9bfa2be2c9214e07e950a0a79`；镜像构建时间与宿主源码提交分别记录。训练源码与发布提交见上文，预处理源码为上述数据代码 commit。

## 3. H200 与 NAS 位置

旧版 Gray1 数据和实验结果 NAS 根目录：

```text
\\10.2.26.26\902_data\0-项目\13-专项\2-数据集\LEO模型量化测试数据\20260922_21目标灰度量化数据
```

H200 挂载路径：

```text
/mnt/ywang-nas/0-项目/13-专项/2-数据集/LEO模型量化测试数据/20260922_21目标灰度量化数据
```

包含 `yolo_full` 的原始全集快照：

```text
\\10.2.26.26\902_data\0-项目\13-专项\2-数据集\21类目标R通道图像归档\snapshots\auto-landmark-Dataset-20260922-161828
/mnt/ywang-nas/0-项目/13-专项/2-数据集/21类目标R通道图像归档/snapshots/auto-landmark-Dataset-20260922-161828
```

| 资产 | H200 当前路径 | NAS 路径 | 状态 |
|---|---|---|---|
| 原始 `yolo_full` | `/data2/ai-i-chenshiwen/auto_landmark/Dataset/yolo_full` | 包含于原始全集快照 `auto-landmark-Dataset-20260922-161828` | `VERIFIED`；删除仍需 owner 授权 |
| 原始全集索引 | `/data3/ywang/yolo-gray1-datasets/manifests/yolo-full-index-20260917.json` | `21类目标R通道图像归档/yolo-full-index-20260917.json` | `ASSET_VERIFIED`；保留 H200 工作副本 |
| M 规模选择索引 | `/data3/ywang/yolo-gray1-datasets/manifests/yolo-medium-index-20260917.json` | `20260922_21目标灰度量化数据/datasets/yolo-medium-index-20260917.json`；Gray1 数据集内另有相同哈希的 `source_manifest.json` | `ASSET_VERIFIED`；保留 H200 工作副本 |
| 当前旧版 Gray1 数据集 | `/data3/ywang/yolo-gray1-datasets/pose21-gray1-1280-m-v1` | `datasets/train.tar` | `ASSET_VERIFIED` |
| cal462 标定集 | `/data3/ywang/quantitize-platform/data/shared_data/cali/pose21_gray1_50e_cal462_v1/cali` | `datasets/cal462.tar` | `ASSET_VERIFIED` |
| eval462 快速评估集 | `/data3/ywang/quantitize-platform/data/shared_data/test/pose21_gray1_50e_eval462_v2` | `datasets/eval462.tar` | `ASSET_VERIFIED` |
| 训练结果 | `/data3/ywang/yolo-gray1-train/runs/pose21_gray1_m_50e_20260917_113127` | `results/training/` | `ASSET_VERIFIED` |
| 6,600 张 PT 推理结果 | `/data3/ywang/yolo-gray1-train/runs/pose21_gray1_m_50e_test_eval_20260918_212142` | `results/inference_6600.tar` | `ASSET_VERIFIED` |
| 最终量化结果 | `/data3/ywang/quantitize-platform/data/output_data/20260920_063903_Pose21_Gray1_weight-only_accuracy_valida` | `results/quantization/` | `ASSET_VERIFIED` |

### 3.1 原始全集快照校验

`auto-landmark-Dataset-20260922-161828` 对整个
`/data2/ai-i-chenshiwen/auto_landmark/Dataset` 建档，因此不仅包含 `yolo_full`，
也保留同目录下的其他历史数据。已验证事实如下：

| 项目 | 结果 |
|---|---:|
| 源文件 | 136,579 个 |
| 源文件内容 | 897,366,345,935 bytes |
| `tar.zst` 分片 | 32 个 |
| 归档大小 | 864,109,237,775 bytes |
| 完整 `SHA256SUMS` | PASS |
| `zstd -t` | 32/32 PASS |
| train/val/test 恢复抽样 | PASS |
| 独立校验完成时间 | 2026-09-23 05:45:17 CST |

快照内无 `.partial`。详细分片与恢复证据见
[归档完成报告](../class-21/operations/runs/backup/raw-dataset/auto-landmark-Dataset-20260922-161828/completion-report.md)。

### 3.2 历史 R-only 过渡实验

2026-09-03/04 的 21 类三通道 R-only 过渡实验不属于当前 Gray1 基线。其旧
`best.pt` 和数据选择清单已归档到：

```text
\\10.2.26.26\902_data\0-项目\13-专项\2-数据集\21类目标R通道图像归档\历史实验\20260903-04_21类R-only过渡实验
```

权重 SHA-256 为
`94125be741cc05bb2df3f6d17d15fb07b182fb74011c2f5ab2669be1c8f89e2f`。
图片和标签通过选择清单引用原始全集快照，不重复占用 NAS；本地
旧工作区下的 `E:\codex\quantitize\data` 已于 2026-09-25 清理；这是删除时的历史路径，并非现用目录。本机根目录更名见[记录](../archive/2026-10-workbench-rename/README.md)。详见
[完成报告](../class-21/operations/runs/backup/historical-r-only/20260925-local-data-cleanup/completion-report.md)。

### 3.3 2026-10-16 完整训练/PT 推理镜像

```text
\\10.2.26.26\902_data\0-项目\13-专项\4-代码\20261016_H200_docker_images\yolo_gray1_1280_train_predict
/mnt/ywang-nas/0-项目/13-专项/4-代码/20261016_H200_docker_images/yolo_gray1_1280_train_predict
```

| 文件 | 内容与身份 |
|---|---|
| `yolo-gray1_1280-train-predict-20261016.tar` | `docker save` 未压缩完整镜像，11,364,463,104 bytes；SHA-256 `ae6a9919c3204990af34f06918739d3ea97454560d43962365c1c14243d0ccb1` |
| `yolo-runtime.tar.gz` | 完整 Dockerfile 构建所需离线运行时，5,344,887,926 bytes；SHA-256 `e1e2f5eb38d40ab81c8075ef45624da04d24420e42807f78c5e9f8903e8af683` |
| `source.bundle`、`source.tar.gz` | 发布分支完整 Git 历史与 `c981f5b...` 受跟踪源码快照，不包含未知诊断文件或凭据 |
| `Dockerfile`、`BUILD_IMAGE.md`、`README.md` | 完整构建定义、依赖文件及恢复说明 |
| `records/source-state.json`、`records/image-inspect.json`、`records/archive-verification.json` | 源码和镜像身份、TAR 层内容校验及 NAS 回读结果 |
| `records/obsolete-images-before.json`、`records/cleanup.json` | 清理前镜像身份与实际清理结果 |
| `records/smoke/` | 原构建 CPU/GPU 烟测脚本和日志、NAS 导入后 CPU 复测日志；不包含正式业务模型 |
| `SHA256SUMS`、`VERIFIED` | 根目录保留逐文件校验清单和备份完成标记 |

大文件按 `.partial → 内容校验 → NAS 独立回读 SHA-256 → 原子改名` 发布。镜像另检查配置哈希等于完整 image ID、所有层 diff ID，并从 NAS 执行 `docker load`，回读后检查 ID 和 CPU 推理。验证使用现有 H200 daemon；未执行新主机隔离恢复，因此此资产属于 `ASSET_VERIFIED`，整套平台仍为 `PARTIAL`。目录只有在 `VERIFIED` 存在且 `sha256sum -c SHA256SUMS` 全部通过时才可用于恢复。

2026-10-16 本目录全量 `SHA256SUMS` 检查通过，已写 `VERIFIED`。随后非强制清理无容器引用的旧训练 v1、v2、v1-predictor-gray1 三个标签，两个旧镜像 ID 均已删除，`records/cleanup.json` 状态 `PASS`。API/Web 容器 ID 和启动时间未变；其 `/health` 正常，但清理前已观察到 API `/ready` 的 NVML 初始化错误，未在本任务修复，见 `records/smoke/service-checks.json`。

后续按用户要求将五个 JSON 记录及烟测目录归入 `records/`，已更新 README、SHA256SUMS 相对路径和 VERIFIED 的清单摘要。记录内容逐文件哈希不变；镜像、依赖包和源码归档未改动，沿用本次完整验收的哈希。恢复时仍从备份根目录执行校验命令。源码快照与 bundle 保持原发布内容，其内部文档对应原布局，当前 NAS 布局以根目录 README 为准。

恢复命令：

```bash
set -euo pipefail
IMAGE_NAS='/mnt/ywang-nas/0-项目/13-专项/4-代码/20261016_H200_docker_images/yolo_gray1_1280_train_predict'
findmnt -t cifs -T "$IMAGE_NAS" -o SOURCE,FSTYPE
test -r "$IMAGE_NAS/VERIFIED"
(cd "$IMAGE_NAS" && sha256sum -c SHA256SUMS)
IMAGE='yolo-gray1:1280-train-predict-20261016'
EXPECTED='sha256:79b0086e39b83c8a3bbb76bfe2df1d8683bd443a6bef19ee7d2e69f096033635'
if docker image inspect "$IMAGE" >/dev/null 2>&1; then
  test "$(docker image inspect "$IMAGE" --format '{{.Id}}')" = "$EXPECTED"
fi
docker load --input "$IMAGE_NAS/yolo-gray1_1280-train-predict-20261016.tar"
test "$(docker image inspect "$IMAGE" --format '{{.Id}}')" = "$EXPECTED"
```

从 Dockerfile 重建还需要 `ubuntu:22.04` 基础镜像、该发布提交的完整 `src/Yolo_fine/` 与 `.dockerignore`；把运行时包恢复到仓库 `docker/yolo-runtime.tar.gz` 后校验再构建。直接 `docker load` 不需要单独准备这些构建材料。数据集、正式权重和 API/Web 不包含在此镜像归档中，详见 [H200 运行环境](02_H200训练测试推理与量化运行环境.md)。

### 3.4 2026-10-16 预处理、API/Web 镜像与总索引

共享根目录为 `\\10.2.26.26\902_data\0-项目\13-专项\4-代码\20261016_H200_docker_images`。Linux 对应 `/mnt/ywang-nas/0-项目/13-专项/4-代码/20261016_H200_docker_images`；本次已核对来源为 `//10.2.26.26/902_data`、类型 CIFS。

保留第 3.3 节训练目录和全部原文件不变，新增三个同级资产目录。根目录新增 `README.md`、`INDEX.json`、`SHA256SUMS`、`VERIFIED`；集合级服务状态记录位于 `records/backup-20261016.json`。

| 资产目录 / 镜像 TAR | 镜像 ID 前缀 | TAR 字节数 | TAR SHA-256 |
|---|---|---:|---|
| `yolo_gray1_data_pipeline/yolo_gray1_data_pipeline.tar` | `ae238a8a9099` | 361,152,512 | `7baa5c1050ee25768b182383453da581194f4f2ffdbc830aff56f0d23a69c011` |
| `quantitize_platform_api/quantitize_platform_api.tar` | `f633b01c2f85` | 11,658,015,232 | `76ce1e923dc8ff12584472cfab47dd8174438c98081d6091cecaad6d550b13ed` |
| `quantitize_platform_web/quantitize_platform_web.tar` | `7aef48747d3f` | 191,891,968 | `74c7a66c2f255668e22f676f5e101f55f07d3c5f2dbf9741adab51d3e0fc167e` |

每个新目录包含未压缩 `docker save` TAR、`source.bundle`、`source.tar.gz`、`Dockerfile`、`README.md`、`SHA256SUMS`、`VERIFIED`。镜像身份、源码身份、已安装 Python/系统包版本和验证证据统一位于 `records/`。预处理源码固定 `f147cee93c83876348205ca268eb930010192116`，API/Web 宿主源码固定 `b1ac5ea84e0374d9bfa2be2c9214e07e950a0a79`；后者晚于镜像构建，不能将宿主源码 commit 当作原镜像构建身份。平台未跟踪 `tools/` 未打包；凭据、数据集、正式模型、scratch 和 `data/output_data` 未打包。

验收包括 TAR 配置哈希等于完整 image ID、全部层 diff ID、NAS 原子发布后独立回读 SHA-256、逐文件清单及 Git bundle 校验；三个新目录各 9 项清单通过，原训练目录 22 项通过，无 `.partial` 或 Docker 临时归档残留。包版本从无网络、只读、限额 CPU 临时容器收集，未调用 GPU。新镜像归档未执行 `docker load` 或新主机隔离恢复，状态为 `ASSET_VERIFIED`。

恢复时先在共享根目录核对 `VERIFIED` 与 `sha256sum -c SHA256SUMS`，再进入所需资产目录核对该目录完成标记和完整清单。确认目标标签不存在或完整 ID 相同后，用 `docker load --input <该目录镜像.tar>` 导入，回读 image ID；完整 ID 和具体命令以该目录 README 与 `records/image-inspect.json` 为准。API/Web 还需要从源码快照恢复挂载的 `apps/api`、`pipeline`、`apps/web` 及匹配的部署配置，模型与数据引用独立 NAS 归档。

完整镜像已经包含已安装运行依赖；`records/runtime-dependencies.json` 是版本盘点，不能替代独立 wheel/deb 包仓库。API Dockerfile 所需基础标签 `ywang/yolo-gray1-quant:ultralytics-8.3.98-v6` 仍缺失；本轮未修复或验证从 Dockerfile 完全离线重建。API/Web 容器 ID、启动时间均不变，备份前后 health 均 HTTP 200，API ready 均 HTTP 503；本轮未诊断或修复该既有异常。

### 3.5 工程批次与后增资产覆盖

500 张工程批次的 10.13 时点备份位于 NAS 共享前缀 `\\10.2.26.26\902_data\0-项目\13-专项` 下的 `2-数据集\LEO模型量化测试数据\20261013_21类目标FPGA上板测试数据`。原记录为 6,077 个源文件、16,766,791,565 bytes；后来增加的 `5-cam2000/sample_index.csv` 不在这次备份中。详细凭证见[500 张报告](../class-21/reports/current/20261012_Pose21_500张原始测试集生成与测试记录.md)，本次没有重新校验 NAS。

后加逐层 trace 不因基础 bundle 已归档就自动视为已备份。10.18 最新 main 与 Git 分支 bundle 也不属于 10.16 镜像快照；[源码记录](../agent/topics/REPOSITORIES.md)只记录 H200 本地分支归档，尚无其异机副本验收。本节维护覆盖边界，不复制实验指标。

### 3.6 2026-10-20 本机 PT 临时 NAS 副本

从本机 `E:\codex\CL-vision-workbench` 将三份 PT 按原相对路径复制到用户指定的 `\\10.2.26.26\902_data\0-项目\13-专项\2-数据集\pt文件备份(临时)\CL-vision-workbench-20261020\`。目标快照新建且不覆盖旧文件；每份先写 `.partial`、比对源文件 SHA-256，核对临时文件后同目录改名，再从 NAS 独立回读。Windows 直连共享目录，本轮未访问 H200 或使用 GPU。

| 原相对路径 | 字节数 | SHA-256 | 身份 |
|---|---:|---|---|
| `docs/archive/2026-09-gray1-step15-checkpoint/best.pt` | 199,108,234 | `292cb13d75cdd571053b92e07f91b03e9e80c0b76b691b022065f445ff1f527f` | 历史 Gray1 Step 15 |
| `docs/archive/2026-09-local-artifact-cleanup/model/yolov8x-pose-p6_2class-r-only_baseline_df30b9fb.pt` | 199,005,770 | `df30b9fbc1cb5eb48e9efadb53b2dfa6e52375255625438f8dc68ee27cc5c8e0` | 历史 2 类 R-only |
| `outputs/class-21/20260920_063903_Pose21_Gray1_weight-only_accuracy_valida/input/model.pt` | 199,108,106 | `622cff54db472b61c85db03648b36c119f030eacbd4cd87cf5d2dda67e69d6a7` | 正式 Gray1 训练权重的任务输入副本 |

快照内三份 PT 共 597,222,110 字节，独立回读大小与 SHA-256 全部一致。`MANIFEST.json`、`SHA256SUMS` 和 `VERIFIED.json` 记录范围与结果；manifest SHA-256 为 `2696bea88298c26663131e8ed1cd81c51a43352b3a9057316bdaec936b3e9a33`。本机原件仍保留，临时 NAS 目录不等于独立异地备份；尚未做隔离恢复，不升级整套恢复状态。

## 4. NAS 结构与清单

当前环境恢复优先使用 10.16 统一目录；早期系统快照、旧 v1 环境仅作历史追溯。目录中的大文件不重复合并：

```text
20261016_H200_docker_images/
├── README.md
├── INDEX.json
├── SHA256SUMS
├── VERIFIED
├── records/
│   └── backup-20261016.json
├── yolo_gray1_1280_train_predict/
│   ├── yolo-gray1_1280-train-predict-20261016.tar
│   ├── yolo-runtime.tar.gz
│   ├── source.bundle / source.tar.gz
│   ├── Dockerfile / BUILD_IMAGE.md / README.md
│   ├── SHA256SUMS / VERIFIED
│   └── records/
├── yolo_gray1_data_pipeline/
├── quantitize_platform_api/
└── quantitize_platform_web/
```

后三个资产目录结构一致：`<目录名>.tar`、`source.bundle`、`source.tar.gz`、`Dockerfile`、`README.md`、`SHA256SUMS`、`VERIFIED` 和 `records/`。`records/` 保存 `image-inspect.json`、`source-state.json`、`runtime-dependencies.json`、`archive-verification.json`；根目录清单保护总索引、集合记录及各资产清单/标记，不能代替每个资产目录的完整文件校验。

以下为独立的数据与结果归档，未复制进 10.16 镜像目录：

```text
20260922_21目标灰度量化数据/
├── datasets/
│   ├── train.tar
│   ├── train.tar.sha256
│   ├── cal462.tar
│   ├── cal462.tar.sha256
│   ├── eval462.tar
│   └── eval462.tar.sha256
├── results/
│   ├── training/
│   ├── inference_6600.tar
│   ├── inference_6600.tar.sha256
│   └── quantization/
│       ├── input_symlinks.tar
│       └── input_symlinks.tar.sha256
└── README_BACKUP_AND_RESTORE.md
```

### 4.1 TAR 归档

| 归档 | 内容 | 普通文件数 | 字节数 | SHA-256 |
|---|---|---:|---:|---|
| `datasets/train.tar` | 完整旧版 Gray1 数据集，包含 train/val/test | 53,597 | 15,507,005,440 | `ce3e93d15311ff75fc46e28eaf2a551de055f459cf01ba2c8638ca21979088d5` |
| `datasets/cal462.tar` | 462 张量化标定图片 | 462 | 268,922,880 | `25de513bc763d13d7f72ba311e988fcffa6919d25adf41405802ce3e0269ffee` |
| `datasets/eval462.tar` | 462 张 PNG、462 个标签 TXT、5 个复现/校验文件 | 929 | 266,229,760 | `dd31eded2b20878aaeb7cbdab7892fe28efd93fc483e668e4d860916be5b97cd` |
| `results/inference_6600.tar` | 完整 PT test 的指标、逐图结果、预测标签及误差明细 | 6,615 | 14,899,200 | `5c6db47b765f0624567d38279c223c30716ba072626a8616bac5031b2dae3b3f` |

表中只统计普通文件，不统计目录条目。`eval462.tar` 有 932 个 TAR member，其中 3 个是目录，普通文件为 `462 images + 462 labels + 5 metadata = 929`。5 个 metadata 是 `SOURCE.md`、`SHA256SUMS`、`prepare_pose21_quant_set.py`、`evaluation-selection.json` 和 `evaluation-distribution.json`。

这些 `.tar` 是未压缩的标准 TAR 归档，恢复时直接解包，不需要 gzip、zstd 或密码。

### 4.2 结果目录

| 目录 | 内容 | 已验证身份 |
|---|---|---|
| `results/training/` | 37 个训练产物，包括日志、配置、`best.pt` 和 `last.pt` | `best.pt` SHA-256 见第 2 节 |
| `results/quantization/` | 227 个量化产物，包括 PT/ONNX/FPGA 回环指标、ONNX/BIN 和交付 ZIP | bundle SHA-256 见第 2 节 |

`results/inference_6600.tar` 不是输入数据集，而是冻结 `best.pt` 对完整 6,600 张 test 图片推理后生成的结果。`results/quantization/input_symlinks.tar` 只保存原任务中 `input/cali` 和 `input/test` 的链接定义，不包含数据内容；真实数据已分别保存在 `datasets/cal462.tar` 和 `datasets/eval462.tar`。该链接归档 SHA-256 为 `fa4e91477d4c7c1051b2a649c8feb9fcf132c38989cd95b548dd164e3354277c`。

## 5. 归档规则

- 大文件流程固定为：写 `<name>.partial`，执行格式/内容测试，计算 SHA-256，在同目录原子改名，最后标记 `ASSET_VERIFIED`。
- 源目录在清点、复制和校验期间必须保持稳定；源持续变化时停止发布。
- 大资产只保存一份，其他 manifest 通过路径、任务 ID 和 SHA-256 引用。
- 源码备份应包含 Git bundle、HEAD、status、diff、未跟踪文件和 remote refs。
- 镜像备份应包含 `docker save` TAR（可按任务要求保留未压缩）、image ID、inspect、repo digest（若存在）、构建来源和内容测试结果。
- 系统备份明确排除 `data/output_data`；任务输出必须像本次一样独立归档。
- `.partial` 不是有效备份，不能用于恢复或作为删除源数据的依据。

## 6. 校验与恢复

### 6.1 Windows PowerShell

```powershell
$nas = '\\10.2.26.26\902_data\0-项目\13-专项\2-数据集\LEO模型量化测试数据\20260922_21目标灰度量化数据'
$archive = Join-Path $nas 'datasets\train.tar'
$restore = 'D:\Pose21_Gray1_restore\training'

Get-Content "$archive.sha256"
Get-FileHash -Algorithm SHA256 $archive

New-Item -ItemType Directory -Force $restore | Out-Null
tar -tf $archive | Select-Object -First 20
tar -xf $archive -C $restore
```

`Get-FileHash` 输出必须与对应 `.tar.sha256` 一致。其他归档只需替换 `$archive` 和 `$restore`。Windows 也可使用 7-Zip 查看或解包，但正式恢复前仍须核对 SHA-256。

### 6.2 Linux/H200

```bash
root='/mnt/ywang-nas/0-项目/13-专项/2-数据集/LEO模型量化测试数据/20260922_21目标灰度量化数据'

(cd "$root/datasets" && sha256sum -c train.tar.sha256)
(cd "$root/datasets" && sha256sum -c cal462.tar.sha256)
(cd "$root/datasets" && sha256sum -c eval462.tar.sha256)
(cd "$root/results" && sha256sum -c inference_6600.tar.sha256)
(cd "$root/results/quantization" && sha256sum -c input_symlinks.tar.sha256)
```

解包旧版 Gray1 数据集和推理结果：

```bash
mkdir -p /data3/ywang/restore/pose21-training
tar -xf "$root/datasets/train.tar" -C /data3/ywang/restore/pose21-training

mkdir -p /data3/ywang/restore/pose21-test-eval
tar -xf "$root/results/inference_6600.tar" -C /data3/ywang/restore/pose21-test-eval
```

训练和量化结果是普通目录，可直接浏览或复制：

```bash
cp -a "$root/results/training" /data3/ywang/restore/
cp -a "$root/results/quantization" /data3/ywang/restore/
```

### 6.3 恢复后验收

```bash
# 当前旧版 Gray1 数据集：53,597 个普通文件
find /data3/ywang/restore/pose21-training/pose21-gray1-1280-m-v1 -type f | wc -l

# PT test 结果：6,615 个普通文件，状态 PASS
run=/data3/ywang/restore/pose21-test-eval/pose21_gray1_m_50e_test_eval_20260918_212142
find "$run" -type f | wc -l
cat "$run/status.env"
python3 -m json.tool "$run/artifacts/summary.json" | head -80
```

解包前建议预留至少 20 GB 空间。不要直接覆盖正在使用的数据集或运行目录，应先恢复到新的空目录验收。

### 6.4 按 10.16 版恢复 Docker 镜像

以下命令用于已授权的新建或隔离 Linux Docker 环境，仅导入指定镜像，不启动服务或申请 GPU。将 `ASSET` 设为第 4 节四个资产目录名之一；镜像配置、数据、模型及宿主驱动需另行恢复。

```bash
set -euo pipefail
BACKUP_ROOT='/mnt/ywang-nas/0-项目/13-专项/4-代码/20261016_H200_docker_images'
ASSET='quantitize_platform_api'
test "$(findmnt -rn -t cifs -T "$BACKUP_ROOT" -o SOURCE)" = '//10.2.26.26/902_data'
test -r "$BACKUP_ROOT/VERIFIED"
(cd "$BACKUP_ROOT" && sha256sum -c SHA256SUMS)
case "$ASSET" in
  yolo_gray1_1280_train_predict)
    IMAGE='yolo-gray1:1280-train-predict-20261016'
    ARCHIVE='yolo-gray1_1280-train-predict-20261016.tar'
    EXPECTED='sha256:79b0086e39b83c8a3bbb76bfe2df1d8683bd443a6bef19ee7d2e69f096033635' ;;
  yolo_gray1_data_pipeline)
    IMAGE='ywang/yolo-gray1-data-pipeline:0.2.1'
    ARCHIVE='yolo_gray1_data_pipeline.tar'
    EXPECTED='sha256:ae238a8a9099ef31a6c06ef788d964ec9dcc4875f80958cd765c1614b90892db' ;;
  quantitize_platform_api)
    IMAGE='quantitize-platform-api:latest'
    ARCHIVE='quantitize_platform_api.tar'
    EXPECTED='sha256:f633b01c2f85d1fd39bf01e9b9f556056d03c805529781c8879e3172412e8ed2' ;;
  quantitize_platform_web)
    IMAGE='quantitize-platform-web:latest'
    ARCHIVE='quantitize_platform_web.tar'
    EXPECTED='sha256:7aef48747d3fc4270f85133382ffbcf83ea0c53e831f06978ac777b48743d9e9' ;;
  *) exit 1 ;;
esac
ASSET_ROOT="$BACKUP_ROOT/$ASSET"
test -r "$ASSET_ROOT/VERIFIED"
(cd "$ASSET_ROOT" && sha256sum -c SHA256SUMS)
if docker image inspect "$IMAGE" >/dev/null 2>&1; then
  test "$(docker image inspect "$IMAGE" --format '{{.Id}}')" = "$EXPECTED"
fi
docker load --input "$ASSET_ROOT/$ARCHIVE"
test "$(docker image inspect "$IMAGE" --format '{{.Id}}')" = "$EXPECTED"
```

源码应从同一资产目录的 `source.tar.gz` 恢复到新的空目录，或从 `source.bundle` 离线获取后检出记录的固定 commit。训练快照使用发布提交 `c981f5b...`，平台快照包含当前受跟踪的 apps、pipeline、patches 和 deploy 文件。核对源码身份后再配置宿主挂载；`docker load` 不恢复 API 的 `apps/api`、`pipeline`、shared_data、output_data、scratch，也不恢复 Web 的 `apps/web` 挂载。凭据通过独立安全渠道重新配置，不写入文档或 Git。

恢复验收按镜像身份、源码/模块路径、CPU 固定样本、API/Web health 和 API ready 依次进行。10.16 备份时 API ready 已为 503，应单独排查；镜像导入成功不表示该异常已经修复。GPU 训练或量化测试须另按 RUNBOOK 检查资源与任务授权，不能将归档校验直接视为业务精度或 FPGA 验收。

## 7. H200 清理边界

以下资产已有已验证 NAS 副本，可作为后续清理候选，但删除必须是独立任务，并在抽样解包或隔离恢复后执行：

- 当前旧版 Gray1 数据集及其 train/val/test。
- cal462 和 eval462。
- 正式训练结果与冻结权重。
- 6,600 张 PT 推理结果。
- 最终 weight-only 量化任务结果。

以下资产当前不能依据本次备份删除：

- 原始 `yolo_full` 已有 `VERIFIED` 快照，但它位于其他用户目录；备份状态不能替代 owner 的删除授权。
- 未单独归档的外置索引和源码工作区。
- 当前仍需使用的日期训练/PT 镜像、量化/API/Web 环境、其他未单独归档镜像和 FPGA 实机侧资产。2026-10-16 授权清理仅覆盖被日期镜像替代、无容器引用的旧训练 v1/v2/predictor-gray1 三个标签，实际结果见第 3.3 节 `records/cleanup.json`；不包含全局缓存或其他用户镜像。

## 8. 剩余恢复缺口

- `snapshot-20260902-182606` 是 pre-Gray1 快照。
- `ywang-yolo-gray1-env-20260909_124030` 是较早训练环境快照，不能代表当前全部环境。
- 2026-10-15 确认的 `4-代码/训练平台/yolo-gray1-train/` 属于历史 v1 备份；2026-10-16 当前完整训练/PT 镜像、离线运行时和受跟踪源码已补充到第 3.3 节的新目录。旧目录仍保留，不把其标签标记当作当前镜像身份。
- 当前四类镜像与受跟踪源码已归档，见第 3.3、3.4 节。未知诊断文件、平台未跟踪工具、部署凭据、独立离线构建依赖及 FPGA 发布身份仍需按范围补齐；还需完成 6,600 张量化 ONNX 回归与真实 FPGA 数值闭环。
- 尚未执行从 NAS 到隔离目录的完整恢复、模型加载和固定样本复测，因此不能升级为 `RECOVERY_VERIFIED`。
- 后增资产覆盖见第 3.5 节：工程批次后加索引、逐层 trace 与 10.18 新 main/bundle 均需独立核对；不得把 H200 本地归档当成异机副本。
- 5090 部署与双向接管仍属[03 TODO 计划](03_5090训练测试推理与量化运行环境.md)。模型全量回归和 FPGA 精度是业务验收，不作为灾备资产校验的替代条件。
- 历史任务 [`gray1-partial-20260919-010348`](../archive/2026-09-gray1-blocked-backup-run/README.md) 曾因 `shared_data` 持续变化而在写 NAS 前停止；它不是当前这批已验证备份的状态。

下一步优先在隔离环境导入 10.16 四类镜像，恢复对应源码/部署配置并做固定样本验证，再补齐独立离线构建材料和 API 基础镜像入口。后续按 FP16、量化、软件回环和 FPGA 实机发布追加身份与验收证据。计划资产齐全且引用可解析时才可标记 `COMPLETE`；完成隔离恢复与验收后才可标记 `RECOVERY_VERIFIED`。
