# 第 5 阶段收尾确认

日期：2026-10-19，仅本机文档确认，不代表远端验收。

用户明确确认：“按原位保留方案收尾第 5 阶段”。因此本阶段已完成，不再等待逐项移动表，也不安排物理搬迁。

## 最终处置

- 10.17 整合、10.18 main-only、镜像清理与备份记录，以及原始 JSON、nas-docs 和旧清单全部原位保留。
- 已被替代的 PR 待办、标签状态等通过[因果摘要](README.md)与[证据索引](../../class-21/operations/runs/README.md)标明，不作为当前执行指令。
- rebuild/legacy、task-artifacts 按既定职责保留，不打散任务包；原实验结论、脚本、模型及数据不改写。
- 本确认取代本目录 README.md、before.json、validation.json 中此前“移动清单待补”的阶段状态。上述已封存记录保留原字节，反映确认前的真实过程；它们不再表示当前仍有搬迁待办。

## 实际变更与验证

本次仅更新 archive 与 runs 两份外层索引，新增本确认和独立收尾校验材料。开始时核对第 5 阶段已修改文件及原 SHA256SUMS，无基线漂移；两份索引在修改前保存在 [closeout-before.zip](closeout-before.zip)，逐成员哈希与集合完整性通过后原子发布。

收尾检查结果见 `closeout-validation.json`，新增版本清单为 `SHA256SUMS.closeout`。原 SHA256SUMS、before/after/validation 及原文包保持不变，不覆盖既有审计历史。只检查本次文档链接和原阶段文件身份；未访问 work、目录链接、H200、NAS 或 GPU，未运行任何业务脚本。

无本阶段遗留的移动事项。业务恢复、服务就绪、模型验收等缺口仍按原事实来源保留，不能因文档阶段结束而升级为已验收。未自动进入下一阶段。

## 回滚

先核对 SHA256SUMS.closeout，再按 closeout-validation.json 的修改后哈希检查两份索引。如已有后续编辑，保全现场、不覆盖；否则从 closeout-before.zip 按原相对路径逐文件恢复，并核对修改前哈希。保留本确认与审计作为历史记录；没有目录移动或逆向搬迁，不使用宽泛删除。
