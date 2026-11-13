# 2026-11-10 数据备份资料整理

用户授权在 rebuild 下集中组织备份历史、背景、恢复使用方法、提示词和常用程序。本轮只修改本机文件，不访问 work、NAS 或服务器，不执行归档、解包、GPU、服务或删除任务，不提交和推送。

## 实际范围

- 新建 `rebuild/data-backup/`；编写入口、历史、恢复指南、三个提示词模板和脚本说明。
- 从已知精确路径复制四个脚本、两份原始 Dataset 恢复文档、三份工程集审计文件，共九份；原文件保持原字节与原位置。副本见 [SOURCES.json](../../SOURCES.json)。
- 只修改两个既有导航：`rebuild/README.md` 与 `docs/class-21/operations/runs/backup/README.md`，新增本资料入口。
- 发布 `rebuild/SHA256SUMS.20261110-data-backup`；沿用上一版本的原覆盖路径并加入本资料目录及前一清单。旧清单不改写。

## 原文、校验与边界

[before.zip](before.zip) 保存两个导航的修改前原字节；[before.json](before.json)记录 ZIP 与成员哈希。备份已在编辑前完成 ZIP CRC 和逐成员哈希验证，从 `.partial` 发布。

[validation.json](validation.json)记录本轮副本/原文件身份、两份冻结恢复说明和工程集清单的已知哈希、500 张 manifest/哈希集合一致性、18 份原 5090 证据的校验、文档检查及受保护旧清单状态。全局文档检查原输出见[全局检查](documentation-global.json)，本轮修改文件的限定检查见[限定检查](documentation-scoped.json)。全局历史问题与本轮新链接分开报告，不能把未修复的全库问题写成通过。

本轮 8 份新增/修改说明的链接检查通过；全局检查的 109 处缺失链接全部来自既有 `docs/dataset/csw/dataset3_8_experiment_notes.md`，对应未下载的服务器附件，不属于本轮变更，未改写其原引用。检查器统计链接出现次数，不等同于不同附件数。

本轮没有补收远端缺失的 manifests、records 或日志，也没有把数据和镜像复制进本目录；没有执行脚本行为测试。校验仅证明本地资料的记录范围和字节身份，不认证 NAS 当前介质或完整系统可恢复性。

## 回滚

先用新版 rebuild 清单核对本轮文件，再与 validation 中两个导航的修改后身份比较。若导航已有后续修改，先另行保全并合并，不覆盖。从 before.zip 只恢复本轮修改的两个导航，按 before.json 校验；原始报告和脚本无需恢复。新增资料目录及版本清单保留作审计，不执行宽泛递归删除。

后续补收证据使用[提示词 C](../../PROMPTS.md)，新建日期目录和版本化清单，不覆盖本次校验结论。
