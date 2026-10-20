# 本机资料的 Git 版本管理

Git 用来记录文件版本；暂存区是下次提交的候选集合；提交才会形成可追溯版本；推送才会发送到远程。项目名称为 `CL-vision-workbench`，本机路径为 `E:\codex\CL-vision-workbench`，与 H200 上的平台、训练和预处理源码仓库分开管理。

2026-10-20 早期两次物理更名被 Windows 占用锁阻止；后续已在新目录核对文件和 Git 身份。原审计见[更名记录](archive/2026-10-workbench-rename/README.md)，现状核验见[完成记录](archive/2026-10-workbench-rename/RENAME_COMPLETION.md)。

## 跟踪范围

| 内容 | Git 处理 |
|---|---|
| 文档、原始报告、图片/标签、允许纳入的模型与数据、CSV/XLSX、日志、数据清单和哈希 | 按根 `.gitignore` 白名单进入候选；不改动原始字节 |
| `.bin` 与 `.zip` | 暂时不纳入 Git 或 LFS，文件原位保留；恢复或迁移仍依赖独立副本与原始清单 |
| `.pt` 与 `.onnx` | 不纳入 Git 或 LFS；三份现有 PT 的本地原件保留，临时 NAS 副本与验收见[备份记录](platform/01_NAS备份与环境恢复.md) |
| `docs/archive/2026-09-class21-transition/test5/` | 历史实验附件整目录不纳入 Git 或 LFS；本机文件原位保留 |
| `work`、临时目录、外部依赖、缓存、凭据 | 排除；不扫描 work 或沿链接读取依赖 |
| Python、Shell、PowerShell、JavaScript 等脚本及源码 | 不跟踪，仍原位保留；包括历史、冻结、临时及文档检查脚本 |
| 明确用于展示的 `outputs/fpga_*` 预览图 | 精确排除，不删除；历史 before 对照图、原始工作簿、统计和来源报告保留 |
| 历史原文 ZIP | 原样保留于本机与独立副本，暂不纳入 Git；内部可能封存旧脚本，不拆包或改写证据身份 |
| `.gitignore`、`.gitattributes` | 唯一两份根级版本管理规则，必须跟踪 |

未识别的新格式默认不跟踪；确认是文档、数据或日志后再审查规则。不要用 `git add -f` 把被排除材料硬塞入仓库。Git 忽略不是删除授权，也不表示被忽略材料不重要。

## 大文件与字节身份

本仓库不通过 Git LFS 保存或推送文件。`.gitattributes` 仅保留 `* -text`，关闭自动换行转换，以保护冻结文档、原始数据和旧清单的字节身份。`.pt`、`.onnx`、`.bin`、`.zip` 与历史 `test5/` 附件排除在 Git 之外，移出暂存区不删除本地文件。

克隆本仓库不需要安装 Git LFS；GitHub 仓库不是模型或完整环境备份。推送前仍须检查暂存文件大小、权限与敏感信息；新增模型应先安排独立备份，不用 `git add -f` 绕过排除规则。不要默认建立公开仓库。

## 提交与推送

本机主分支为 `main`，`origin` 目标为 `https://github.com/space-exploration-101/CL-vision-workbench.git`；早期 LFS 配置已从本仓库卸载。初次暂存候选已移出 `.pt`、`.onnx`、`.bin`、`.zip` 与历史 `test5/` 附件。每次提交前审查暂存清单，提交与推送是否完成以 Git 现场状态为准；此前物理更名的核验见[完成记录](archive/2026-10-workbench-rename/RENAME_COMPLETION.md)。Git 提交和远程推送都不能代替独立备份。

在当前实际目录中，可只读检查：

```text
git status --short
git diff --cached --stat
git ls-files
git log -1 --oneline
git remote -v
```

每次推送前复核暂存清单、远程地址、访问权限和 GitHub 单文件限制；提交、配置远程与推送须有明确授权。不要自动提交 H200 上仅同步路径的文档改动，也不要修改其 remote 或部署路径。

## 备份与恢复的区别

这个仓库不是软件源码仓库，也不是完整可执行环境。脚本、依赖、临时材料、`.pt`、`.onnx`、`.bin`、`.zip` 及历史 `test5/` 附件未被跟踪；克隆后文档中部分实现或附件链接可能指向未随 Git 分发的原位文件。它们在本机仍有效，三份 `.pt` 另有临时 NAS 副本；不能为消除链接提示而删除历史引用。

完整的更名前副本保存在项目外 `quantitize-backups`，详情、文件清单与回滚约定见[本轮记录](archive/2026-10-workbench-rename/README.md)。旧审计仍使用旧根路径，按同一相对路径映射到新根；远端部署、NAS、镜像和备份目录名称保持不变。
