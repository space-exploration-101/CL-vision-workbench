# CL-vision-workbench：视觉模型工程资料工作区

这里汇集视觉项目的数据准备、训练、测试、推理、量化、FPGA 交付及环境运维资料，不是正在运行的服务源码仓库。当前主要案例是 21 类 Gray1 模型。

本机目录现为 `E:\codex\CL-vision-workbench`；原 `E:\codex\quantitize` 是历史路径。更名核验及旧路径解释见[完成记录](docs/archive/2026-10-workbench-rename/RENAME_COMPLETION.md)。

本仓库管理文档、允许纳入的原始数据和日志；脚本、依赖、临时材料、work 以及 `.bin`、`.zip` 文件原位保留，暂不纳入 Git。`.pt`、`.onnx` 和指定大型张量文本使用 Git LFS（大文件存储扩展）。首次提交或推送前先读[版本管理说明](docs/VERSION_CONTROL.md)。H200 部署目录及源码仓库名称不随本机目录更名。

**先理解三个概念**：模型是从样例中学到的识别规则；量化是用较少位数表示其中的数值，以节省存储和计算；FPGA 是可配置的计算芯片，生成交付文件后还需要在真实硬件上验证。

## 从哪里开始

| 读者或目标 | 入口 |
|---|---|
| 第一次接手、没有算法背景 | [交接手册](docs/HANDOFF.md)，再按问题查[文档导航](docs/README.md) |
| 理解图像、神经网络与量化 | [项目基础知识](docs/class-21/knowledge/01_YOLO_Gray1与量化基础.md)、[详细入门](docs/reference/VISION_MODEL_QUANTIZATION_FPGA_PRIMER.md) |
| 查看模型、数据与实验结论 | [21 类项目](docs/class-21/README.md) |
| 使用 Agent 协助工作 | [人类任务指南](docs/AGENT_GUIDE.md) |
| 日常诊断、服务和备份 | [平台运维入口](docs/operations/README.md)，只展开所需任务卡 |
| 自动化 Agent | [AGENTS.md](AGENTS.md) → [简要上下文](docs/agent/CONTEXT.md) → [任务索引](docs/agent/RUNBOOK.md) |
| 恢复服务或重建环境 | [平台说明](docs/platform/README.md)、[恢复手册](docs/reference/RECOVERY.md)、[重建入口](rebuild/README.md) |
| 查历史变更原因 | [归档入口](docs/archive/README.md) |
| 本次整理的最终范围、阶段交付、校验和回滚 | [最终交接](docs/archive/2026-10-final-handoff/README.md) |

## 最近有证据的状态

以下业务状态汇总依据 2026-10-18 及之前的记录。2026-10-20 更名仅核对 H200 文档引用与源码身份，未重新验证服务、NAS 或 GPU。

- 21 类 Gray1 模型采用单通道灰度输入；最终方案为权重 INT8、计算中间值 FP16（W8A16）。
- 6,600 张原始模型测试和 462 张量化对比已完成；500 张工程集已有结果，但包含与训练集重叠的场景。
- 真实 FPGA 精度、6,600 张量化模型全量回归尚无通过记录。
- 三个源码仓库的最新 main 位于 H200 的 `/data3/ywang/main-worktrees/<仓库名>`；旧运行目录保留原提交。源码更新不等于服务已部署。
- 四类运行镜像及对应源码已有 10 月 16 日 NAS 校验副本；整套隔离恢复仍未完成。10 月 16 日记录的 API 就绪检查异常仍待现场复核。

详细身份、证据日期和待办见[交接手册](docs/HANDOFF.md)。Web 的已记录入口为 <http://10.2.29.180:8088>，不代表本次已验证在线。

每类动态事实只在[来源表](docs/platform/SOURCES.md)指定文档维护；本页是摘要。事实疑点见[待核对项](docs/platform/OPEN_QUESTIONS.md)，不因文档整理合并冲突或提高验收状态。

## 目录用途

| 目录 | 定位 |
|---|---|
| [docs/](docs/README.md) | 当前知识、操作入口、实验依据及明确分区的历史归档 |
| [rebuild/](rebuild/README.md) | 当前重建路径；过期环境材料在 `legacy/` |
| [tools/](tools/README.md) | 辅助脚本，包含需绑定原任务才能使用的冻结脚本 |
| [task-artifacts/](task-artifacts/README.md) | 已有任务材料快照，不能当作最新源码 |
| [outputs/](outputs/README.md) | 正式附件、分析来源与过程文件、展示预览按文件分类；不以目录名判断可删除或可再生成 |
| `.tmp/` | 临时工作材料，仅按明确任务访问；不作为默认校验或事实检索入口 |
| `work/` | 临时外发材料；本次及默认索引、检索均排除，不能据此判断项目状态 |

文档不保存明文密码。原始副本与本次整理记录见[整理记录](docs/archive/2026-10-documentation-consolidation/README.md)。
