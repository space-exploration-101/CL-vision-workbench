# CL-vision-workbench 更名与 Git 初始化

2026-10-20，依据用户确认，将本机资料工作区命名为 `CL-vision-workbench`，更新本机及 H200 上指向该工作区的有效引用，并建立仅管理文档、数据和日志的 Git 仓库。本记录独立于既有七阶段整理验收，不覆盖旧证据。

## 当前执行状态：物理更名待完成

项目名称、Git/LFS 配置与文件跟踪规则已落实。两次 `Rename-Item` 均被 Windows 的目录占用锁拒绝；**实际根目录仍为 `E:\codex\quantitize`，目标目录尚不存在**。未强行停止进程、调整安全全局设置、复制 work 或创建 junction。根 README、HOSTS 及 H200 控制端说明区分实际与目标路径，不能宣称更名已完成。

关闭占用旧目录的终端、编辑器和项目会话后，从项目外的 PowerShell 执行：

```powershell
Set-Location -LiteralPath 'E:\codex'
if (Test-Path -LiteralPath 'E:\codex\CL-vision-workbench') {
    throw '目标已存在：先保全并核对，不覆盖'
}
Rename-Item -LiteralPath 'E:\codex\quantitize' -NewName 'CL-vision-workbench'
```

接续 Agent 应先读新目录的 AGENTS、CONTEXT、RUNBOOK 和本记录，现场确认新旧目录及 `.git` 身份，再保存本轮之后的文档新版本；仅更新明确标记“待更名”的本机入口及 H200 单份 CONTEXT，新增独立完成记录和核验，不改本目录已封存结果。不重做迁移、不全局替换历史路径、不提交或推送。目录未解除占用前停止重试。

## 范围与基线

- 旧根目录：`E:\codex\quantitize`；目标根目录：`E:\codex\CL-vision-workbench`。
- 独立完整副本：`E:\codex\quantitize-backups\20261020-100158-before-CL-vision-workbench`。已逐文件 `.partial` 写入、回读校验、发布，再复核源与副本 SHA-256；4,720 个普通文件、269 个目录、5 项链接定义、6,316,052,916 字节。原始复制完成于北京时间 10:03:29。
- `work` 未读取或复制；依赖链接仅记录定义，不复制目标；副本不认证 ACL、备用数据流或文件系统原子快照。此次是独立副本，不依赖旧备份的当前可用性。
- `before.json`、`before.zip` 保存本轮将修改的本机文档和 H200 文档原文；`baseline/` 保存备份清单与位置。新版本与预期哈希不符时停止，不覆盖现场。
- 不重算实验，不修改模型、张量、BIN、表格、冻结脚本、包内清单及历史验收。不使用 GPU、不访问 NAS、不重启服务。

## 路径解释

| 类型 | 处理 |
|---|---|
| 本机当前工作区根路径 | 目标为 `E:\codex\CL-vision-workbench`；占用锁解除前实际保持旧根，相对路径不变 |
| 历史报告、审计、清单中的 `E:\codex\quantitize` | 保留原文；同一相对路径可在新根目录定位，但已删除的历史文件不会因此恢复 |
| `E:\codex\quantitize-backups` | 真实备份目录保持原名，不做字符串替换 |
| `/data3/ywang/quantitize-platform`、main-worktrees、scratch、镜像与远程仓库名称 | 属于独立部署/源码身份，保持不变 |
| 包内旧路径、ZIP 成员与受保护清单 | 保持原字节；原迁移映射继续有效 |

H200 检查限定为三个原运行/源码目录与三个 main 工作区的文档和配置入口。排除数据、archive、evidence、runs、legacy、work、隐藏目录及依赖/符号链接；不声称扫描服务器全盘。初查 137 个文件，仅原平台 `docs/agent/CONTEXT.md` 含需更新的控制端路径。远端各仓库 HEAD 不变，不提交、不推送。

## Git 边界

Git 是文件版本记录；Git LFS 将大文件内容单独保存，Git 中保存其内容标识。本机只初始化仓库和暂存候选，不创建提交、远程地址或推送。使用规则见[版本管理说明](../../VERSION_CONTROL.md)。

原文中的冻结脚本引用继续保留，但脚本不进入新仓库。Git 克隆不是完整环境恢复副本；完整本机副本仍由外部备份承担。历史证据 ZIP 原样保留，可能包含当时封存的实现原文，不将其当作现用脚本分发。

## 核验与回滚约定

最终执行结果以本目录独立新增的验证清单为准；本文件中的目标不提前代表目录更名、远端更新或 Git 暂存已完成。

已完成的分项：H200 单份文档发布并回读通过，另外 136 个受检文件、六个仓库 HEAD 均不变；本机首轮 122 份文档链接校验通过，冻结链接继续使用 9 项精确映射。Git 候选中的 1,399 份可扫描文本未命中本轮常见凭据模式；未解包或检查大数据内容，这不是隐私/公开发布许可。原始数据规模约 5 GB，远程 LFS 容量仍需确认。

Git 初始化第一次由沙箱用户创建空元数据，当前用户随后遇到所有者不匹配。纠正所有者被系统拒绝后，没有放宽全局安全检查；经权限审批，将该空元数据保全到 `E:\codex\quantitize-backups\20261020-1010-git-initialization-attempt`，再由当前用户 admin 独立初始化成功。该目录不是文件备份，也不改变前述完整副本的清单。

结果文件分工：`h200-final.json` 是第一版目标路径发布后的校验；后续 `h200-pending-update.json` 和 `h200-pending-final.json` 记录更名受阻后更正为“实际旧路径、目标新路径”的版本，不覆盖前一轮记录。`links-before-rename.json` 和 `links-final-pending.json` 记录文档校验；`after.json` 固定本轮最终文档身份；`file-validation-pre-rename.json` 是首次原文件身份核验，`file-validation.json` 是最终差异清单；`git-validation.json` 记录暂存、排除及 LFS；`validation.json` 汇总实际结果，`SHA256SUMS` 封存本轮证据。未列入范围的外部系统不作验收结论。

1. 回滚前先比较本轮 after 哈希，若已有后续修改，先独立保全，不覆盖。
2. 文档可从 `before.zip` 按路径恢复；备份中的本机文件可按 `files.json` 校验。不要整包覆盖后续数据。
3. H200 只恢复本轮修改的文档，使用其独立远端原文副本，并核验当前哈希；不回退源码提交、服务或数据目录。
4. 根目录回滚需关闭占用它的程序，确认旧目录名未被重用，再经文件系统权限审批改回。不要创建混淆新旧目录的 junction。
5. Git 配置与新审计文件可以保留作证据；如需撤销初始化，先确认没有新提交或后续暂存，不自动删除 `.git`。
