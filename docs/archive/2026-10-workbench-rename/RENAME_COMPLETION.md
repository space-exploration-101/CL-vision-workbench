# 工作区物理更名完成记录

2026-10-20 10:34（Asia/Shanghai）现场核对。本页接续[更名受阻时的原始记录](README.md)；原记录及其 `SHA256SUMS` 保留原字节，不追改当时的结论。

## 本机核对

- `Get-Location` 与 `git rev-parse --show-toplevel` 均指向 `E:\codex\CL-vision-workbench`；旧目录 `E:\codex\quantitize` 不存在，新目录与 `.git` 存在。
- 根 Git 索引仍有 3,396 个路径；没有 HEAD 提交或 remote。更名前的 `after.json` 中，本次涉及的七份文档及原记录 `README.md` 共八项 SHA-256 在修改前全部匹配。
- 更名前的独立副本 `E:\codex\quantitize-backups\20261020-100158-before-CL-vision-workbench` 仍存在；本次只核对其路径存在，未重做完整副本的逐文件校验。

## 路径更新

- 本机更新根 `AGENTS.md`、根 `README.md`、`docs/VERSION_CONTROL.md`、`docs/agent/CONTEXT.md`、`docs/agent/topics/HOSTS.md`、`docs/agent/topics/REPOSITORIES.md` 与 `docs/archive/README.md`，只更正当前入口的目录状态及链接。
- 通过 `ssh H200` 直连，仅更新 `/data3/ywang/quantitize-platform/docs/agent/CONTEXT.md` 的控制端路径两行。发布前旧文件 SHA-256 为 `4eebda8816adeb21984982086f4723717e72de587485fd6142cf73b6e9d26cc1`；新文件由同目录 `.partial` 校验后原子改名发布，回读 SHA-256 为 `04bd4f364dc9822cba62fb5fc6e88024634e3c99fa92e4b1c924951968bb7873`。
- 远端回滚副本为 `/data3/ywang/documentation-backups/20261020-CL-vision-workbench/physical-rename-completion/CONTEXT.md`，回读哈希与旧文件一致。原平台 HEAD 仍为 `b1ac5ea84e0374d9bfa2be2c9214e07e950a0a79`；该文档改动未提交，临时 `.partial` 已不存在。

## 边界与回滚

- 历史报告、归档清单、包内旧路径、备份根目录和 H200 部署/源码身份仍按原证据解释；没有做全局替换。本次未访问 GPU、NAS 或服务，也未提交、设置 remote 或推送。
- 若需回滚文档，先核对当前哈希并保全后续改动，再按原记录的 `after.json` / `before.zip` 或远端上述独立副本恢复。物理目录回退需另行核对占用与路径冲突，不由本记录自动执行。
- 本轮已核对更名后的入口文件与 Git 身份，未重新执行整库逐文件、远端服务或环境验收。首次提交、remote 和 LFS 推送仍是独立后续任务。

## 验收

- `git diff --check`：通过；本次七份本机文档改动为 11 行新增、11 行删除。
- `python -B tools/check_documentation_safe.py`：PASS，检查 122 份文档、665 条活跃直链；9 条冻结链接按既有精确映射单列，原链接未改写。检查范围和七份已更新文档哈希见[独立验证清单](completion-validation.json)。
