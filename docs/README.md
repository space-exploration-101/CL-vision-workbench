# 文档导航

面向人类读者。第一次接手先读[交接手册](HANDOFF.md)，其中先解释模型、训练、量化和硬件验收的关系。自动化 Agent 从[专用入口](agent/README.md)按需读取。

| 需要了解什么 | 文档 |
|---|---|
| 项目做什么、当前进度、谁负责哪个环节 | [HANDOFF](HANDOFF.md) |
| 怎样给 Agent 一个可验证的任务 | [AGENT_GUIDE](AGENT_GUIDE.md) |
| 21 类模型、图像与数据、量化 | [class-21](class-21/README.md) |
| 地球、月球两类数据集与实验过程 | [Earth_Moon](Earth_Moon/README.md) |
| 共用运行环境、备份恢复、5090 TODO 计划 | [platform](platform/README.md) |
| 网络的层、输入输出与硬件布局 | [架构](class-21/architecture/README.md) |
| 某次实验到底证明了什么 | [报告](class-21/reports/README.md) |
| 查报告引用的模型、测试结果和硬件参考文件 | [正式附件](../outputs/class-21/README.md) |
| 日常平台诊断、服务和备份 | [operations](operations/README.md) → 对应任务卡 |
| 模型相关任务模板、工具与既有执行证据 | [class-21 operations](class-21/operations/README.md) |
| 查每类事实的唯一来源及证据日期 | [来源表](platform/SOURCES.md)，疑点见[待核对项](platform/OPEN_QUESTIONS.md) |
| 故障恢复、性能分析、基础教材 | [reference](reference/README.md) |
| 为什么改成今天的方案 | [archive](archive/README.md) |
| 整理后的目录职责、各阶段交付、最终快照及回滚 | [最终交接](archive/2026-10-final-handoff/README.md) |

## 如何理解“当前”

当前说明描述最近有证据的状态，不保证服务器此刻仍相同。每个状态要带观察日期和来源；计划、单项文件校验、软件测试、硬件实测是不同结论。

reference 讲通用方法，operations 选日常运维任务，rebuild 组织完整恢复与构建材料。平台事实在 platform 及来源表指定文档维护；待实施的 03 明确标记 TODO，不混入已验收能力。历史结果留在原报告和执行记录，入口只保留摘要。

`Earth_Moon/` 独立保存地球、月球两类报告及原始笔记，附件本地保存范围见其索引。`class-21/reports/current/` 保存仍支撑21类现用模型的报告，按日期写成的报告不因时间过去自动失效。原始/派生附件统一放在根目录的 `outputs/class-21/`；`operations/runs/` 保存执行凭证；`archive/` 和 `rebuild/legacy/` 保存已经被替代的方案。附带的二进制、日志、旧脚本用于复现，不属于当前操作说明。

维护约定见 [AGENTS.md](AGENTS.md)；本次整理没有访问 `work/`。
