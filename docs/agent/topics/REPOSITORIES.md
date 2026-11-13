# 源码版本与工作区

H200 分支整理证据日期：2026-10-18；5090 源码初始化与 H200 提交复核：2026-10-20，见下方独立小节。

## 最近验收

- 三个项目仓库已 fetch 全部远端跟踪分支并 prune 失效 refs。本地仅保留 main，均跟踪 origin/main：平台 `d206ea6f184ca1773ce10e4c0c0b90204d07a7c0`，训练 `232dfeb1229a67a4f2dd6cd8c2b173041fe7814e`，预处理 `f147cee93c83876348205ca268eb930010192116`。平台 PR #3 已合并，main 包含昨日整合提交。
- 最新 main 的干净源码工作区统一位于 `/data3/ywang/main-worktrees/<仓库名>`。原有所有工作区保留原 HEAD 和文件内容，以 detached HEAD 固定；原生产平台仍为 `b1ac5ea...`，原训练目录仍为 `c981f5b...`。不要把 main 已同步理解为原运行目录已部署新代码。
- 删除 5 个本地分支，包括三个仓库的 5090_env、平台整合分支与旧 snapshot。旧 refs（含旧 main）已保存为 `archive/20261018-before-main-only/<原分支名>` 标签，完整 bundle 和 PASS 审计位于 `/data3/ywang/git-branch-archives/20261018-main-only/`；本次未写 NAS。
- 未修改远端分支、未重启服务或使用 GPU，API/Web 容器身份和启动时间不变。详细验证与回滚见 [仅保留 main 完成记录](../../class-21/operations/runs/20261018-h200-main-only.md)。5090 迁移分支当时已删除；10.20 恢复采用固定运行提交，未新建分支。

开发使用 main 工作区；服务诊断使用原运行目录。恢复 10.16 镜像时使用匹配的当日源码快照，不能自动混入更新后的 main；详见[运行环境](../../platform/02_H200训练测试推理与量化运行环境.md)。

## 5090 源码初始化：2026-10-20

用户授权在 `ubuntu22` 的 `/home/ubuntu22/CL01_02` 建目录并克隆代码。三个仓库均已从 H200 的 `main` 与运行 `HEAD` Git bundle 克隆，工作区干净、`git fsck --full` 通过；[完整执行与哈希证据](../../class-21/operations/runs/20261020-5090-cl01-02-bootstrap.md)。本次没有 fetch GitHub，main 是现场 H200 本地引用的副本。

| 目录（相对 `/home/ubuntu22/CL01_02`） | 当前固定 HEAD（detached） | 保留 main / origin/main |
|---|---|---|
| `quantitize-platform` | `b1ac5ea84e0374d9bfa2be2c9214e07e950a0a79` | `d206ea6f184ca1773ce10e4c0c0b90204d07a7c0` |
| `yolo-gray1-train` | `c981f5b8a7a4f3c8a111c63812d0c74f5fe8b5f2` | `232dfeb1229a67a4f2dd6cd8c2b173041fe7814e` |
| `yolo-gray1-data-pipeline` | `f147cee93c83876348205ca268eb930010192116` | `f147cee93c83876348205ca268eb930010192116` |

各 `origin` 为 `git@github.com:space-exploration-101/<仓库名>.git`；5090 GitHub SSH 连通失败，尚未验证该主机私有仓库权限。bundle 保留在工作根目录的 `source-bundles/`，不依赖在线 GitHub 即可重建本次固定源码；未复制凭据。平台运行提交不是 main 的祖先，bundle 已单独包含运行 HEAD。

初始化时仅记录了 H200 未提交/未跟踪状态，未恢复运行材料。10.20 后续[依赖与资产恢复](../../class-21/operations/runs/20261020-5090-assets-restore.md)进一步核对了实际挂载、模块和补丁；运行相关 119 个受跟踪文件的规范化内容全部相同，11 个文件仅 CRLF/LF 不同。两端原始与规范化 SHA-256 均保留，未重写源码以消除换行差异。

平台未跟踪的两个 `tools/prepare_pose21_quant_set.py` 相关工具与训练的 `Dockerfile.predictor-chfix`、`smoke_predictor_warmup.py` 已按精确路径保存在 5090 `assets/h200-supplements/` 并校验，未覆盖进三仓或执行。H200 的控制端路径文档改动不作为 5090 运行事实。镜像、数据和模型恢复进度只在平台 03 维护；本次仍未部署服务或执行 GPU 任务。

## 本机资料仓库

2026-10-20 本机项目名称确认为 `CL-vision-workbench`，独立初始化 Git，仅纳入文档、原始数据和日志；物理目录随后更名为 `E:\codex\CL-vision-workbench`，没有关联远程、提交或推送。它不是上述三个源码仓库之一，也不是完整重建包。规则见[版本管理](../../VERSION_CONTROL.md)，更名核验见[完成记录](../../archive/2026-10-workbench-rename/RENAME_COMPLETION.md)。

本轮通过 H200 直连复核上述六个源码/运行目录的 HEAD，与本页 10.18 记录一致；仅更新原平台 `docs/agent/CONTEXT.md` 的 Windows 控制端路径，留下未提交的文档改动。其他五个工作区未改，未更新服务或业务状态。精确范围与原文见[更名记录](../../archive/2026-10-workbench-rename/README.md)。后续物理更名后再次只更新同一控制端说明，回读哈希与回滚副本见[完成记录](../../archive/2026-10-workbench-rename/RENAME_COMPLETION.md)。
