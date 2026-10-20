# 源码版本与工作区

证据日期：2026-10-18。本地文档整理没有重新访问服务器。

## 最近验收

- 三个项目仓库已 fetch 全部远端跟踪分支并 prune 失效 refs。本地仅保留 main，均跟踪 origin/main：平台 `d206ea6f184ca1773ce10e4c0c0b90204d07a7c0`，训练 `232dfeb1229a67a4f2dd6cd8c2b173041fe7814e`，预处理 `f147cee93c83876348205ca268eb930010192116`。平台 PR #3 已合并，main 包含昨日整合提交。
- 最新 main 的干净源码工作区统一位于 `/data3/ywang/main-worktrees/<仓库名>`。原有所有工作区保留原 HEAD 和文件内容，以 detached HEAD 固定；原生产平台仍为 `b1ac5ea...`，原训练目录仍为 `c981f5b...`。不要把 main 已同步理解为原运行目录已部署新代码。
- 删除 5 个本地分支，包括三个仓库的 5090_env、平台整合分支与旧 snapshot。旧 refs（含旧 main）已保存为 `archive/20261018-before-main-only/<原分支名>` 标签，完整 bundle 和 PASS 审计位于 `/data3/ywang/git-branch-archives/20261018-main-only/`；本次未写 NAS。
- 未修改远端分支、未重启服务或使用 GPU，API/Web 容器身份和启动时间不变。详细验证与回滚见 [仅保留 main 完成记录](../../class-21/operations/runs/20261018-h200-main-only.md)。5090 迁移分支当前不存在，实施时从确认后的 main 在独立工作区新建。

开发使用 main 工作区；服务诊断使用原运行目录。恢复 10.16 镜像时使用匹配的当日源码快照，不能自动混入更新后的 main；详见[运行环境](../../platform/02_H200训练测试推理与量化运行环境.md)。

## 本机资料仓库

2026-10-20 本机项目名称确认为 `CL-vision-workbench`，独立初始化 Git，仅纳入文档、原始数据和日志；物理目录随后更名为 `E:\codex\CL-vision-workbench`，没有关联远程、提交或推送。它不是上述三个源码仓库之一，也不是完整重建包。规则见[版本管理](../../VERSION_CONTROL.md)，更名核验见[完成记录](../../archive/2026-10-workbench-rename/RENAME_COMPLETION.md)。

本轮通过 H200 直连复核上述六个源码/运行目录的 HEAD，与本页 10.18 记录一致；仅更新原平台 `docs/agent/CONTEXT.md` 的 Windows 控制端路径，留下未提交的文档改动。其他五个工作区未改，未更新服务或业务状态。精确范围与原文见[更名记录](../../archive/2026-10-workbench-rename/README.md)。后续物理更名后再次只更新同一控制端说明，回读哈希与回滚副本见[完成记录](../../archive/2026-10-workbench-rename/RENAME_COMPLETION.md)。
