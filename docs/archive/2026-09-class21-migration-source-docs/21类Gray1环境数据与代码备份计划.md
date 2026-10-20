# 21 类 Gray1 环境、数据、代码与任务资产备份计划

> 文档状态：滚动归档计划，当前量化平台仍在开发  
> 编制日期：2026-09-19  
> 适用范围：Gray1 数据预处理、训练、独立推理、量化、FPGA 交付  
> 当前结论：可以立即固化已经验证的资产，但不得把部分归档标记为“完整灾难恢复就绪”

## 1. 目标与边界

本计划解决两个不同问题：

1. 在量化平台开发完成前，先保护已经稳定的数据预处理、训练和推理资产，避免 H200 单点丢失。
2. 在后续每个量化里程碑完成时增量归档，使最终恢复包自然形成，而不是等全部开发结束后再追溯现场。

当前源码、镜像、数据、模型和任务产物处于不同成熟度，不能强行打成一个“已完成”快照。归档状态分为：

| 状态 | 含义 |
|---|---|
| `PARTIAL` | 只覆盖部分资产，不具备完整恢复能力 |
| `ASSET_VERIFIED` | 单个资产完成复制、格式检查和身份验证 |
| `COMPLETE` | 计划内所有资产齐全，但尚未做干净环境恢复 |
| `RECOVERY_VERIFIED` | 已从备份完成一次 clean-room 恢复和验收 |

量化开发完成前，顶层快照最高只能是 `PARTIAL`；其中已完成的单个资产可以是 `ASSET_VERIFIED`。

## 2. 当前资产状态

以下身份来自 2026-09-18 至 19 日记录，执行归档前仍需在 H200 只读复核。

| 资产组 | 当前状态 | 归档结论 |
|---|---|---|
| 数据预处理代码 | GitHub 可恢复，commit `f147cee93c83876348205ca268eb930010192116` | 可立即归档 Git bundle 和构建信息 |
| 数据预处理镜像 | `ywang/yolo-gray1-data-pipeline:0.2.1`，ID `sha256:ae238a8a...92db` | 可立即归档镜像 |
| 正式派生数据 | `pose21-gray1-1280-m-v1`，26,794 图像和 26,794 标签，约 15.47 GB | 可立即完整归档 |
| 原始数据 | 约 824 GiB、66,006 项，位于其他用户目录 | 先登记合同；复制需数据 owner 明确授权 |
| 训练代码 | base commit `dd70691faa8eb276301717d3d182b624cb653d14` | 可立即归档 commit；Predictor 工作树需额外保存 |
| 训练镜像 | `...:ultralytics-8.3.98-v1`，ID `sha256:7a291a40...9d03` | 可立即归档 |
| 推理修复 | Predictor 修改及 Docker/smoke 文件尚未提交 | 立即保存 patch、未跟踪文件和恢复说明 |
| 推理镜像 | `...:v1-predictor-gray1`，ID `sha256:edaa6104...6c68c` | 可立即归档 |
| 正式模型 | 50 epoch `best.pt`，SHA-256 `622cff54...6a7` | 可立即永久归档 |
| 独立 test | 6,600 张，运行 `pose21_gray1_m_50e_test_eval_20260918_212142`，PASS | 已有独立 NAS 备份；应登记进统一 catalog |
| 量化平台代码 | H200 commit `c45eb80abbe582a9194cb310a2be66dc5cfbabab` 未在远端分支 | 立即保存 Git bundle |
| 量化 API/Web 镜像 | 当前镜像身份已知 | 可作为开发现场立即归档，但不是最终 Gray1 发布镜像 |
| Gray1 FP16/QDQ/FPGA | 尚未完成 | 后续按里程碑追加 |

现有 NAS 备份不能互相替代：

- `snapshot-20260902-182606` 是 pre-Gray1 量化平台快照；
- `ywang-yolo-gray1-env-20260909_124030` 是较早训练环境快照；
- `20260918_21类识别结果(灰度图padding版本)` 保存独立 test、模型和报告，但不包含完整 train/val、全部镜像和源码恢复材料。

## 3. 建议的长期目录

当前 21 类 Gray1 项目统一使用以下 NAS 根目录。用户已指定父目录
`\\10.2.26.26\902_data\0-项目\13-专项\4-代码`，本计划在其下建立独立的
`21类Gray1` 子目录，避免资产直接散落在父目录：

```text
NAS_ROOT=\\10.2.26.26\902_data\0-项目\13-专项\4-代码\21类Gray1
H200_ROOT=/mnt/ywang-nas/0-项目/13-专项/4-代码/21类Gray1
```

`NAS_ROOT` 是 Windows UNC 路径；`H200_ROOT` 是同一共享目录在 H200 已确认挂载后的预期路径。正式写入前必须用挂载信息和目录身份验证二者确认为同一位置，不能仅凭路径名称推断。

目录结构：

```text
21类Gray1/
├── README.md
├── catalog/
│   ├── asset-index.jsonl
│   ├── lineage.jsonl
│   └── snapshots/
├── sources/
│   ├── yolo-gray1-data-pipeline/<source-id>/
│   ├── yolo-gray1-train/<source-id>/
│   └── quantitize-platform/<source-id>/
├── environments/
│   ├── container-images/<image-id>/
│   ├── runtime-locks/
│   └── host-inventories/
├── datasets/
│   ├── raw/<raw-dataset-id>/
│   ├── prepared/<dataset-id>/
│   └── selections/<selection-id>/
├── models/<model-id>/
├── runs/
│   ├── preprocessing/<run-id>/
│   ├── training/<run-id>/
│   ├── evaluation/<run-id>/
│   └── quantization/<task-id>/
├── releases/<release-id>/
└── recovery/<snapshot-id>/
```

大文件只在资产目录保存一份。任务目录通过 manifest 引用 `dataset_id`、`model_id`、`image_id` 和 SHA-256，不为每个任务重复复制同一镜像、完整数据集或权重。

## 4. 当前可以立即归档的内容

### 4.1 源码和构建上下文

为三个仓库分别保存：

```text
repository.bundle
HEAD.txt
git-status.txt
git-diff.patch
untracked-files.tar.zst
remote-refs.txt
source-manifest.json
```

规则：

- clean 且已推送的仓库仍保存一次 Git bundle，作为 GitHub 不可用时的冷备份；
- `quantitize-platform` 的 `c45eb80...` 尚未出现在远端分支，Git bundle 是 P0；
- 训练仓库 Predictor 修改未提交，必须同时保存 patch 和四个 Docker/smoke 未跟踪文件；
- 不把工作树复制品当作唯一源码备份；后续应提交恢复 commit 或合并正式 commit。

### 4.2 当前已验证镜像

首批归档：

```text
ywang/yolo-gray1-data-pipeline:0.2.1
ywang/yolo-gray1-train:ultralytics-8.3.98-v1
ywang/yolo-gray1-train:ultralytics-8.3.98-v1-predictor-gray1
quantitize-platform-api:latest        # 当前开发基线，不是最终发布
quantitize-platform-web:latest        # 当前开发基线，不是最终发布
```

每个镜像目录保存：

```text
image.tar.zst
image-inspect.json
repo-digests.txt
build-source-ref.json
archive-stream-test.txt
SHA256SUMS
VERIFIED
```

`predictor-chfix` 是中间诊断镜像。默认只保存 Dockerfile、镜像 ID 和构建关系；除非仍需复现故障，不永久保存完整大镜像。

### 4.3 正式派生数据和预处理证据

立即归档完整 `pose21-gray1-1280-m-v1`，包括 train、val、test，而不是只备份 6,600 张 test：

```text
images/{train,val,test}
labels/{train,val,test}
dataset.yaml
manifest.json
source_manifest.json
SHA256SUMS
VERIFIED
verification_report
配置快照和运行记录
独立 acceptance 目录
```

建议按 split 分片归档，避免单个 15 GB 文件损坏影响全部数据：

```text
train.tar.zst
val.tar.zst
test.tar.zst
metadata.tar.zst
```

复用数据集已有 `SHA256SUMS` 和 manifest。新归档文件只在发布时计算一次哈希，不重复扫描整个数据集。

### 4.4 原始数据合同

在没有 owner 授权复制 824 GiB 原始数据前，先保存：

```text
raw-source-contract.json
完整文件索引或权威索引引用
数据 owner
读取位置
数量和容量
类别、split、标签格式
恢复责任和可用性承诺
源 manifest 身份
```

这只能标记为 `SOURCE_REGISTERED`，不能写成 `RAW_BACKED_UP`。以后由 owner 决定完整备份、共享权威副本或第二套等价来源。

### 4.5 正式训练、模型和独立 test

训练运行至少保存：

```text
args.yaml
results.csv
训练日志
数据、代码、镜像引用
best.pt
last.pt
FROZEN.yaml
训练/val 指标摘要
运行状态和资源记录
```

模型同时注册到独立目录：

```text
models/pose21-gray1-m-1280-e50-20260917/
  best.pt
  last.pt
  model-card.yaml
  training-run-ref.json
  dataset-ref.json
  environment-ref.json
  FROZEN.yaml
  VERIFIED
```

独立 test 已有 NAS 副本，本阶段不重复复制。应读取该副本的文件清单，把其路径和身份登记到 `catalog/asset-index.jsonl`，并补充指向正式 model/dataset 的引用。

### 4.6 当前量化开发现场

虽然 Gray1 量化尚未完成，仍应归档当前可复现的开发基线：

```text
quantitize-platform Git bundle
当前 API/Web 镜像
Compose 和安全化 env.example
运行时版本清单
当前接口合同
已知阻塞和失败任务摘要
旧两类/三通道 golden task 引用
```

该组资产必须标记：

```yaml
scope: development_baseline
gray1_quantization_completed: false
production_ready: false
fpga_validated: false
```

## 5. 在 H200 完成后逐步补全的内容

### 5.1 FP16 导出完成后

追加：

- 最终导出代码 commit 和镜像 ID；
- FP16 ONNX、输入输出 ABI、模型检查报告；
- PT/FP16 同集 parity 报告；
- 固定的 evaluation selection manifest；
- 导出命令和日志。

验收后建立 `quantization/<task-id>/checkpoint-fp16.json`，但快照仍为 `PARTIAL`。

### 5.2 QDQ 量化完成后

追加：

- calibration selection manifest；
- calibration 数据集引用，不重复复制正式数据；
- QDQ ONNX；
- 量化配置、provider、runtime 和日志；
- PT、FP16、QDQ 同集指标；
- NaN/Inf、shape、非全零和后处理检查；
- 当前量化 API 镜像的最终归档。

若量化代码或镜像与开发基线不同，创建新 `source-id` 和 `image-id`，禁止覆盖旧资产。

### 5.3 完整 6,600 张量化回归后

追加：

- 全量 evaluation run；
- 与 PT 基线相同口径的 summary、per-image、漏检、误检和像素误差文件；
- 逐类退化分析；
- 预先约定的准入阈值及 PASS/FAIL；
- 量化模型与正式 test 数据集的精确引用。

### 5.4 FPGA BIN 和软件回环完成后

追加：

- FPGA 输入协议、测试包和 roundtrip 结果；
- 权重/BN BIN、offset 表、ABI manifest；
- 逻辑 31 通道到物理 32 通道的证据；
- generate/merge/bundle 的完整日志和状态；
- golden 输入输出样本；
- 最终 bundle。

失败任务只保存配置、日志、关键中间产物和故障结论；大规模 trace/tensor 设置保留期限，除非它们被指定为 golden。

### 5.5 FPGA 实机和发布完成后

追加：

- 板卡、固件、host 程序和 ABI 版本；
- 实机输出、性能、资源、精度和回归报告；
- 可发布 bundle 及其签名/哈希；
- 旧 2 类 golden regression；
- release note 和回滚包。

只有此阶段满足业务阈值时，单个 release 才能标记 `production_ready=true`。

## 6. 每次运行前就要准备的记录

不要等任务完成后再补日志。每个 preprocess/train/eval/quant task 创建时先写：

```yaml
task_id:
task_type:
created_at:
code_refs: []
image_refs: []
dataset_refs: []
model_refs: []
config:
expected_outputs: []
retention_class: permanent|decision|temporary
status: CREATED
```

运行期间追加阶段状态，完成后补充：

```yaml
finished_at:
actual_outputs: []
metrics:
logs:
failures: []
artifact_inventory:
verification:
status: COMPLETED|FAILED|KNOWN_BLOCKED
```

所有任务从一开始就引用固定的代码、镜像、数据和模型身份。禁止使用无法解析到 image ID 的 `latest`、未记录的目录内容或人工记忆。

## 7. 推荐的实际执行顺序

### 阶段 A：冻结目录和命名，不复制大文件

1. 用户确认唯一 NAS 根目录及容量预算。
2. 定义 `source-id`、`image-id`、`dataset-id`、`model-id`、`run-id`、`release-id`。
3. 创建空目录结构、schema 和 catalog。
4. 记录现有分散 NAS 备份的外部引用。
5. 输出 dry-run 文件清单、估算字节数和预计时间。

### 阶段 B：先保护不可替代的小文件

1. 归档三个仓库的 Git bundle、status、diff 和未跟踪文件。
2. 归档配置、manifests、acceptance、训练/test 报告和当前文档。
3. 注册正式模型身份。
4. 验证归档可读后发布对应 `ASSET_VERIFIED`。

### 阶段 C：归档当前大文件

1. 依次归档数据预处理、训练、Predictor、API、Web 镜像。
2. 分 split 归档正式派生数据。
3. 归档正式 best/last 和必要训练现场。
4. 不重复复制已有独立 test 备份，只登记并验证其引用。
5. 每个资产独立 `.partial -> 流测试 -> 哈希 -> 原子发布 -> VERIFIED`。

### 阶段 D：量化开发期间滚动追加

按 FP16、QDQ、小集评估、全量评估、FPGA 软件包、实机六个 checkpoint 追加资产。任何阶段失败都保留真实状态，不制造后续成功产物。

### 阶段 E：形成统一恢复快照

量化和 FPGA 阶段达到预定范围后，生成只包含 manifest、引用和必要发布资产的 recovery snapshot。检查所有引用均能在 NAS/GitHub 解析，再标记 `COMPLETE`。

### 阶段 F：恢复演练

在新目录或隔离主机执行：源码恢复、镜像加载、数据校验、模型推理、训练 smoke、PT/ONNX/QDQ 回归和必要的 FPGA 软件验证。全部通过后才标记 `RECOVERY_VERIFIED`。

## 8. 保留策略

| 资产 | 建议期限 |
|---|---|
| 原始数据或权威源合同 | 永久 |
| 被正式模型使用的 prepared 数据集 | 永久 |
| 正式 best/last、发布 ONNX/BIN/bundle | 永久 |
| 形成决策的训练、评估和量化任务 | 至少 1–3 年 |
| 普通失败任务 | 问题关闭后 90–180 天 |
| 大型 trace、逐层 tensor、临时 BIN | 默认 30–90 天 |
| `/data1` scratch | 7–14 天 |
| 已归档的 `/data3` 任务 | NAS 验证后仍保留约定宽限期 |

删除必须基于 catalog 和引用检查。任何仍被 model、evaluation、quantization 或 release 引用的资产不得删除。

## 9. 当前停止条件

遇到以下情况停止归档或发布：

- NAS 不是 `//10.2.26.26/902_data` 的真实 CIFS 挂载；
- 目标目录已存在或会覆盖历史资产；
- 源码、镜像、模型或数据身份与本计划记录不一致；
- 原始数据复制没有 owner 授权；
- 容量不足或归档会影响共享服务器任务；
- 大文件流测试失败、存在 `.partial` 或 manifest 不完整；
- 任务引用了未登记的 mutable `latest`、目录或配置；
- 部分备份被错误标记为完整恢复快照。

## 10. 本轮备份完成条件

量化开发未完成时，本轮合理目标不是 `RECOVERY_VERIFIED`，而是：

```text
源码不可替代状态已保护
+ 当前正式镜像已归档
+ 正式 prepared 数据已归档
+ 正式模型和训练/test 证据已登记
+ 量化开发基线已固化
+ 后续 task 有固定的增量归档模板
= PARTIAL snapshot with verified assets
```

完成这些工作后，即使 H200 出现故障，也能恢复数据预处理、训练、独立推理和量化开发起点；尚未完成的 Gray1 量化、FPGA 和发布资产则按后续 checkpoint 继续补全。
