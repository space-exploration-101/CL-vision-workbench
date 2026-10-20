# 21 类项目正式附件

本目录保存[当前报告](../../docs/class-21/reports/current/README.md)引用的模型与实验文件。模型是训练得到的识别规则；张量是多维数值数组；BIN 是按硬件约定排列的二进制数据。它们是可复查的证据，不等于真实 FPGA 已通过验收。

| 入口 | 内容 |
|---|---|
| [最终量化任务](20260920_063903_Pose21_Gray1_weight-only_accuracy_valida/README_bundle.md) | 模型、指标、BIN 与后续逐层导出 |
| [单图文本导出](20261013_agaier_047_view_18_txt/) | `agaier_047_view_18` 的文本参考文件 |
| [500 张工程测试附件](pose21-raw-gray-test500-20261012-v1/) | 选图证据、PT/ONNX 结果和 NAS 备份凭证；[相机格式说明](pose21-raw-gray-test500-20261012-v1/cam2000/CAMERA_BIN_ABI.md)解释字节布局 |
| [网络架构演示文稿](YOLO网络架构图_Pose21_Gray1_21类_更新版.pptx) | 模型分层示意；结合[逐层报告](../../docs/class-21/reports/current/20260925_Pose21_Gray1_单图逐层输入输出导出试验.md)使用 |
| [FPGA 回传附件组](FPGA_logs/) | output.dat、映射、CSV/XLSX；对应[500 张报告](../../docs/class-21/reports/current/20261012_Pose21_500张原始测试集生成与测试记录.md)中的 10.18/19 分析，不等于完整硬件验收 |
| [逐层输出与 PPT 页码对照](20260928_Pose21_Gray1_逐层输出与PPT页码对照.csv) | 逐层报告引用的原始对应关系 |

按报告中的精确路径读取，不需要遍历整个目录。附件包中的 README、manifest（文件清单）和 SHA-256（内容指纹）描述生成时的身份；其中 `before_*` 文档是包内历史快照，不能覆盖当前说明。新增参考不自动进入旧 ZIP 或旧 NAS 备份。

2026-10-18 本目录由 `docs/class-21/reports/output/` 整体迁入，仅更新外层 README/AGENTS，包内结构、内容和原校验清单保持不变。包内若记录旧本机路径，按[迁移映射](../../docs/archive/2026-10-artifact-relocation/README.md)定位，不批量改写原始证据；H200/NAS 路径不变。旧模型过渡实验另见[历史归档](../../docs/archive/2026-09-class21-transition/README.md)。

2026-10-19 将原 reports/current 下的 FPGA_logs 整组及页码对照 CSV 迁入本目录，8 个附件文件内容保持不变。包内 FPGA_logs/README.md 中指向 `../20260925_...md` 的旧相对链接沿用生成时位置，请改从本页的[逐层报告](../../docs/class-21/reports/current/20260925_Pose21_Gray1_单图逐层输入输出导出试验.md)入口访问；这是历史路径说明，不代表文件缺失。详见[本阶段映射与回滚](../../docs/archive/2026-10-platform-docs/README.md)。

## 冻结旧引用的精确解释

| 冻结位置或旧路径 | 解释为现用位置 | 边界 |
|---|---|---|
| 旧 `docs/class-21/reports/output/<相对路径>` | `outputs/class-21/<相对路径>` | 仅本机路径前缀映射，包内相对结构不变；不重写远端路径或 ZIP 成员 |
| 旧 `docs/class-21/reports/current/FPGA_logs/<文件>` | `outputs/class-21/FPGA_logs/<文件>` | 整组 7 个文件已迁入，不重复移动 |
| 旧 `docs/class-21/reports/current/20260928_Pose21_Gray1_逐层输出与PPT页码对照.csv` | [现用 CSV](20260928_Pose21_Gray1_逐层输出与PPT页码对照.csv) | 单个原始 CSV 已迁入，数据不改写 |
| `outputs/class-21/FPGA_logs/README.md` 内 `../20260925_Pose21_Gray1_单图逐层输入输出导出试验.md` | [当前逐层报告](../../docs/class-21/reports/current/20260925_Pose21_Gray1_单图逐层输入输出导出试验.md) | 原链接字面解析到 outputs/class-21 下不存在的报告；冻结原文保留，活跃入口使用右侧链接 |

这里的正式附件身份取决于报告引用及冻结清单，而非扩展名。例如 FPGA_logs 中名称带 preview 的 XLSX 仍随正式包保护，不能归为可删除图片。outputs 同级 `fpga_*` 的来源工作簿、报告历史版本、统计和脚本另按[材料分类](../README.md#原位材料分类索引)原位保留，不替换本目录证据。

第 6 阶段未修改包内内容，身份复核见[收尾核验](../../docs/archive/2026-10-attachment-closeout/README.md)。可再生成性、业务精度、FPGA 实机与 NAS 恢复不在本轮验证范围。
