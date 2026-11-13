# 环境恢复与重建

面向恢复环境的工程师。**恢复镜像**是加载已经打包的软件；**重新构建**是从源码、基础镜像和依赖重新组装，两者所需材料不同。

截至已有 2026-10-16 备份和 10.18 源码记录，四类镜像及对应受跟踪源码已归档；整套恢复状态仍为 `PARTIAL`。本轮仅更新本地文档，没有重新执行恢复。

1. 先读 [CURRENT_STATE](CURRENT_STATE.md)，确认已覆盖资产和剩余缺口。
2. 按 [REBUILD_GUIDE](REBUILD_GUIDE.md) 选择导入镜像或重新构建。
3. [rebuild-manifest.yml](rebuild-manifest.yml) 是当前恢复摘要索引，不是自动安装清单。
4. 数据备份的历史、背景、解包使用说明、预存提示词和常用脚本快照统一从 [data-backup](data-backup/README.md) 进入；原始报告与维护脚本仍保持原位。
5. 当前本地清单为 [SHA256SUMS.20261110-nas-paths](SHA256SUMS.20261110-nas-paths)。原 SHA256SUMS 与此前版本均原样保留；旧清单按对应阶段副本验证，不能用来判断本轮文档仍与旧版相同。校验只证明本地文件身份，不证明 NAS 内容或恢复成功；本轮见[NAS 路径复查与文档记录](data-backup/audit/20261110-nas-paths/README.md)，上一轮见[资料整理记录](data-backup/audit/20261110/README.md)。

过期的环境清单、Dockerfile 和验证脚本已归入 [legacy](legacy/README.md)，不可直接用于当前环境。原始完整目录及旧清单保存在[整理前文档包](../docs/archive/2026-10-documentation-consolidation/README.md)。日常服务故障先看[恢复手册](../docs/reference/RECOVERY.md)。

本目录组织完整恢复／构建的材料与验收，不另维护环境或备份详细事实。日常平台运维从[operations](../docs/operations/README.md)进入；通用判断方法在[reference](../docs/reference/README.md)。本机目录并不包含完整镜像或离线依赖，实际材料按[唯一来源表](../docs/platform/SOURCES.md)获取和核对。
