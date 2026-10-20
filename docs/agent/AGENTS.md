# Agent 文档范围

继承 [文档规则](../AGENTS.md)。CONTEXT 和 RUNBOOK 保持为短索引；专题事实放 topics，执行步骤放 tasks。新增主题前检查是否已有对应文件，避免复制长篇知识库。

每份任务卡写清触发条件、授权范围、必要输入和验收。链接采用相对路径；任务卡不能覆盖根 AGENTS 的共享主机、GPU、凭据和 work 隔离边界。

任务卡不复制提示词或动态状态表；模板统一见 [AGENT_GUIDE](../AGENT_GUIDE.md#统一模板入口)，工具身份见[总索引](../../tools/README.md)。

事实分工按[来源表](../platform/SOURCES.md)，BACKUPS 等专题只作按需入口，不要求任务先读全部平台说明。文档整合按 [DOCUMENTATION](tasks/DOCUMENTATION.md)执行，疑点登记后保持未验收状态。
