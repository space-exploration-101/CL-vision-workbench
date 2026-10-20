# 如何使用 Agent 维护量化平台

> 模板编排：2026-10-19；仅本机文档整理，不代表远端验收。

面向让 Agent 协助排查、开发、备份和维护的同事。先读[交接手册](HANDOFF.md)；Agent 执行从根目录 [AGENTS.md](../AGENTS.md) 开始。

<a id="统一模板入口"></a>

## 1. 统一模板入口

**提示词模板**回答“怎样向 Agent 描述任务”，填写目标、输入、范围与授权；**任务卡**回答“执行时遵循什么步骤和门禁”。本页统一索引模板，执行细节以 [RUNBOOK](agent/RUNBOOK.md)选中的卡片为准；当前事实只查[来源表](platform/SOURCES.md)，二者不各自维护状态表。

| 要描述的任务 | 唯一模板位置 | 执行任务卡 |
|---|---|---|
| 所有任务共同信息 | 本页下方共同骨架 | [RUNBOOK](agent/RUNBOOK.md) |
| 服务或任务诊断 | [诊断模板](#服务或任务诊断) | [DIAGNOSTICS](agent/tasks/DIAGNOSTICS.md) |
| 本机文档修改 | [文档模板](#本地文档更新) | [DOCUMENTATION](agent/tasks/DOCUMENTATION.md) |
| PT/ONNX 比较 | [评估模板](#固定数据上的-ptonnx-对比) | [EVALUATION](agent/tasks/EVALUATION.md) |
| FPGA 逐层比较 | [FPGA 模板](#fpga-逐层比较) | [EVALUATION](agent/tasks/EVALUATION.md) |
| 安全重启 | [重启模板](#安全重启) | [SERVICE](agent/tasks/SERVICE.md) |
| 归档、恢复、新数据任务 | [专用模板](class-21/operations/prompts/21类Gray1_NAS归档与恢复资料精简_AGENT提示词.md)（保留原位） | [BACKUP](agent/tasks/BACKUP.md)，新数据再选 [EVALUATION](agent/tasks/EVALUATION.md) |

模板只保存一份正文，不为单个模板增加目录层级。原 operations/prompts 是专用模板存放位置，本页是统一选择入口；不把通用模板复制过去。

### 共同骨架

写清目标、范围、资源和验收；检查服务、改代码、写备份、删除源文件是不同的授权范围。已有授权内应继续完成验证，需要扩大范围时才说明缺口。

```text
先读 AGENTS.md、docs/agent/CONTEXT.md 和 docs/agent/RUNBOOK.md，
再按任务读取对应近期报告；默认检索排除 work/。

目标：<具体结果>
范围：<本机/H200、目录、服务>
允许操作：<只读/修改/备份/启动任务/删除及其前提>
资源：<CPU/内存/磁盘预算，是否允许 GPU、是否已分配>
验收：<指标口径、状态、文件身份或恢复证据>

开始说明检查内容、是否使用 GPU、写入范围。
按已有授权完成；需要扩大范围时说明原因。
结束报告实际变更、验证结果、产物和未完成项。
```

提示词与日志不放密码。`work/` 仅在明确指定其中材料时读取，不能作为判断项目状态的依据。

## 2. 任务对应的资料

| 任务 | 资料 |
|---|---|
| 服务、GPU、任务排障 | [上下文](agent/CONTEXT.md)、[操作手册](agent/RUNBOOK.md) |
| 理解模型与数据 | [知识库](class-21/knowledge/README.md)、[架构](class-21/architecture/README.md) |
| 量化精度 | [9 月完整报告](class-21/reports/current/20260920_Pose21_Gray1_训练测试标定与量化完整报告.md) |
| FPGA 逐层对接 | [单图逐层导出](class-21/reports/current/20260925_Pose21_Gray1_单图逐层输入输出导出试验.md) |
| 500 张测试和相机数据 | [10 月测试记录](class-21/reports/current/20261012_Pose21_500张原始测试集生成与测试记录.md) |
| 备份与恢复 | [NAS 备份与环境恢复](platform/01_NAS备份与环境恢复.md)、[恢复手册](reference/RECOVERY.md) |
| 重建与性能 | [重建指南](../rebuild/REBUILD_GUIDE.md)、[性能报告](reference/PERFORMANCE.md) |

`CONTEXT` 是短摘要，`RUNBOOK` 链接本任务的小型操作卡，实验报告给出结果依据。Agent 按需展开，不必一次读完全部人类教程。9 月初系统快照、两类性能数据和旧激活 QDQ 方案都有适用范围，不能替代后续 21 类结论。

## 3. 常用提示词

### 服务或任务诊断

```text
只读检查 H200 平台与任务 <task_id>。
核对 health/ready、实际阶段、runner/子进程、日志和资源。
区分计算中、等待 GPU、失败，以及进程退出但状态未更新。
结合 GPU 连续采样、显存、进程 owner 与时长判断，不启动或重启任务。
```

### 本地文档更新

```text
根据当前有效报告更新 <文档路径>，核对证据日期和资产身份。
只修改本地文档，默认排除 work/；修正旧路径并验证链接。
区分历史观察和当前状态，不连接服务器，不把未复核状态写成今日已确认。
```

### 固定数据上的 PT/ONNX 对比

```text
使用 <dataset/selection manifest>、<PT哈希>、<ONNX哈希> 同图对比。
核对 Gray1 输入、尺寸、归一化和统计口径；记录镜像 ID、provider。
报告总体、逐类、负样本误检、bbox/关键点像素误差和差异样例。
业务阈值执行前确定；没有阈值时只报告结果，不宣称业务达标。
资源预算 <CPU/内存/GPU授权>，结果写新目录，不覆盖已发布数据。
```

集合重叠与独立性按[数据说明](class-21/knowledge/02_21类目标数据与输入预处理.md)核对，不从模板推断。CPU/GPU 与 provider 应在任务输入中明确。

### FPGA 逐层比较

```text
使用 <单图参考manifest> 与 <FPGA回传目录> 比较。
先核对模型/图片身份、输入布局、有效 lane、补零、字节序和 dtype，
再按依赖顺序比较各层全部输入输出，记录首个偏离层和预设容差。
区分 packed-u12 相机 BIN、FP16 T32 网络入口和检测输出。
本次只做比较，不重新量化，不以软件参考生成代替实机验收。
```

### 安全重启

```text
授权重启 H200 平台 API/Web。
按 RUNBOOK 查任务列表和 runner/worker；有运行或 waiting_gpu 任务则报告，
不授权中断这些任务。核对 Compose 服务名后 restart。
验证容器、health/ready 和历史任务查询；不重建镜像，不用 down -v。
```

### 备份与清理

填写[归档与恢复专用模板](class-21/operations/prompts/21类Gray1_NAS归档与恢复资料精简_AGENT提示词.md)，不在此维护第二份正文。步骤与门禁查 [BACKUP](agent/tasks/BACKUP.md)；模板或历史记录不构成删除授权。

## 4. 判断结果是否可靠

- 证据有日期、主机、模型与数据身份，区分现场观察和历史记录。
- 任务完成同时检查预期阶段、错误和产物；业务精度另按阈值验收。
- health 与 ready 分别检查；GPU 空闲依据连续采样、显存和进程 owner。
- PT/ONNX 同图、软件回环、逐层软件参考、FPGA 实机分别说明。
- 备份有覆盖清单、哈希、格式测试和恢复依据。
- 无证据时不得提高验收状态；具体未决事项查[事实来源与待核对项](platform/OPEN_QUESTIONS.md)，不在模板维护缺口清单。
