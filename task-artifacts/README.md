# 任务材料快照

此处保留 gray1-20261016 任务中的镜像发布与训练仓库材料，作为当日构建/交付来源。嵌套仓库的 README 和 AGENT.md 描述该快照，不是本项目当前全局指令。

## 本地快照的完整性限制

2026-10-19 只读核对 `gray1-20261016/train-repo/RELEASE_SHA256SUMS`：950 条记录中，5 项与本地文件一致、10 项内容不匹配、935 项未包含在本地快照。该目录本地只有 17 个文件，不能把这份发布清单当作本地材料集合已完整通过的证明；未包含不等于已确认丢失，也不能据此推断 NAS 副本异常。

不匹配项为 docker/docker-compose.yml、docker/yolo-runtime.tar.gz.sha256、README.md、AGENT.md、run-gray1-training-smoke.sh、docs/human/02_RUN_TRAINING.md、docs/human/03_TRAINING_CONFIGURATION.md、docs/human/04_RESTORE_ENVIRONMENT.md、docs/agent/WHERE_TO_FIND_THINGS.md、docs/agent/BACKUP_LOCATIONS.md。

本轮不判断差异产生原因，不修改上述文件或原清单、不补齐或重打包。恢复时按平台说明选择匹配的发布资产，不能单独依赖此局部快照。基线保全和验证见[整理记录](../docs/archive/2026-10-platform-docs/README.md)。

当前运行环境见[平台 02](../docs/platform/02_H200训练测试推理与量化运行环境.md)，最新源码见[源码身份](../docs/agent/topics/REPOSITORIES.md)。本次保持快照内容，避免破坏其复现关系。
