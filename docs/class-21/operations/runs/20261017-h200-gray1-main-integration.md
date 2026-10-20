# H200 Gray1 main 整合分支交付记录

> 本文保存 10.17 交付时点。后续平台 PR #3 已合并，三仓 main 与工作区状态见[10.18 完成记录](20261018-h200-main-only.md)；下文未创建 PR 的描述不再是当前待办。

日期：2026-10-17。用户授权从远程 main 选择性整合 H200 功能，生成 commit 并推送，由用户在 GitHub 网页合并。

## 提交与远端结果

- 仓库：`space-exploration-101/quantitize-platform`。
- 基线 main：`4bec9ee9f0432dea98df2b03bd297b9b682902be`。
- 新分支：`integrate/h200-gray1-fixes`。
- 已提交并推送：`c40b71c078de12bea2fa0b94924ec3d0f141c453`，`fix: integrate Gray1 BIN and roundtrip pipeline repairs`。
- 远端回读新分支 SHA 与提交一致，main 仍为上述原提交，整合工作区干净。
- 独立工作区：`/data3/ywang/quantitize-platform-integration`，生产目录仍在 `b1ac5ea...`，未跟踪 tools/ 保留。

15 个文件包含 L43、runner BIN 参数、回环命名/API/Web 兼容、逐层导出及测试/文档。保留 main 的 CI、文档结构和默认测试数量，不引入 .bak、部署快照或二进制。

## 验证

无 GPU、无网络、独立限额临时容器内 37 项 CPU 测试通过。冻结 ONNX SHA-256 为 `031def2070a1c2183e62ce5713831bc69cd50dee3a22c10d0ffaff6580d91a02`；L43 2,176,000 bytes 与旧交付包逐字节一致；`taiwan_276_view_38.png` 完整导出 182 层、510 个 tensor，全部导出哈希通过。文档生成检查和 git diff --check 通过。

固定样本产物位于 `/data3/ywang/quantitize-platform-integration-validation`，未纳入 Git。验证证据已随 commit 放在远程仓库 `docs/evidence/integration/2026-10-17-h200-gray1-fixes.md`。未执行 GPU 全流程、全量精度或 FPGA 实机验收；未部署新代码、重启服务或更改 NAS 备份。

## PR 状态与建议说明

分支已推送，PR 尚未创建：H200/本地无 gh、无现有 API 登录，浏览器工具返回 No browser is available。没有进行自动合并。

创建入口：https://github.com/space-exploration-101/quantitize-platform/pull/new/integrate/h200-gray1-fixes

Base：`main`；Compare：`integrate/h200-gray1-fixes`。

建议标题：`fix: integrate Gray1 BIN and roundtrip pipeline repairs`

建议正文：

```markdown
修复 main 的 BIN 生成调用与输入回环结果展示，并迁入独立逐层导出工具。

- 补齐 L43 图结构系数生成，兼容 4/6/21 类输出识别；runner 传入 ONNX 和预处理模式。
- 统一 fpga_input_roundtrip_eval 阶段，兼容旧 manifest、结果目录和 API 响应，修复 Web overlay 路径。
- 加入 T32 逐层导出、测试、使用说明和不可变验证记录。
- 保留 main 的 CI/文档结构，不引入旧 .bak、部署快照、二进制或测试数量默认值变更。

验证：37 项 CPU 回归通过；冻结模型 L43 与原交付包逐字节一致；固定单图导出 182 层/510 tensor，全部哈希通过；文档生成及 diff 检查通过。

未执行 GPU 完整量化、全量精度回归或 FPGA 实机验收；未修改 H200 生产服务。详细证据见 docs/evidence/integration/2026-10-17-h200-gray1-fixes.md。
```
