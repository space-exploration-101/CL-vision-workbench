# 工具身份总索引

本页唯一维护根 tools 与 docs/class-21/operations/tools 的逐项身份。根目录没有 operations/tools；不为统一名称新增目录、移动或复制工具。工具是执行实现，不是授权书。本轮只静态审阅，所有原脚本内容和位置不变。

## 版本与阅读依据

- **Q**：[9 月 Gray1 报告](../docs/class-21/reports/current/20260920_Pose21_Gray1_训练测试标定与量化完整报告.md)，20260918 选择规则与 Gray1 模型，不代表 6,600 张量化全量回归。
- **T**：[10 月报告](../docs/class-21/reports/current/20261012_Pose21_500张原始测试集生成与测试记录.md)，pose21-raw-gray-test500-20261012-v1 及阶段产物；脚本可能使用迁移前布局，不保证顺序重跑兼容。
- **C**：[输入合同](../docs/class-21/knowledge/02_21类目标数据与输入预处理.md)，Gray1/net1280/cam2000 与对应 grayprep/profile，不是任意相机协议。
- **B**：[备份记录](../docs/class-21/operations/runs/backup/README.md)，历史时点；最新覆盖只查[平台 01](../docs/platform/01_NAS备份与环境恢复.md)。
- **G**：[10.18 完成记录](../docs/class-21/operations/runs/20261018-h200-main-only.md)，历史分支与工作区，不代表 5090 已部署。

精确模型/镜像/数据哈希按原报告核对，不在这里维护第二份状态表。

## 工具身份与使用入口

**M 维护工具**用于当前文档维护；**R 可复用候选**具有参数接口，但跨版本兼容性未在本轮验证；**F 冻结工具**绑定模型、数据合同或实验口径；**O 已完成一次性脚本**仍按冻结证据保护，不删除、不自动重跑。有参数不等于通用维护承诺。

| 唯一文件 | 身份 | 用途与版本依据 | 风险与授权要求 |
|---|---|---|---|
| [check_documentation_safe.py](check_documentation_safe.py) | M | 当前唯一文档校验执行入口 | 本地受控扫描；拒绝 work、临时/依赖目录、越界及重解析点；输出报告仅新建，冻结映射单列 |
| [test_check_documentation_safe.py](tests/test_check_documentation_safe.py) | M | 上述工具的隔离单元测试 | 只执行此测试文件，不发现或导入业务测试，不创建真实 work 或外部依赖 |
| [check_documentation.py](check_documentation.py) | F | 旧文档检查实现，保留原字节；仅供新入口复用纯文本解析函数 | 默认旧遍历含 .tmp 且 junction 隔离不足，不再直接执行；旧阶段审计也不重跑 |
| [archive_dataset_to_nas.py](../docs/class-21/operations/tools/archive_dataset_to_nas.py) | R | B；参数化分片归档 | NAS 写入、压缩与空间占用；须精确路径、挂载及写入授权，不授予删除 |
| [verify_dataset_snapshot.py](../docs/class-21/operations/tools/verify_dataset_snapshot.py) | R | B；配套快照格式校验 | 读取源/NAS、外部校验程序及 nas-root partial 扫描；只读也须完整范围授权 |
| [prepare_pose21_quant_set.py](prepare_pose21_quant_set.py) | F | Q；确定性标定/评估集合 | 复制数据、写清单；固定规则，授权新输出 |
| [test_prepare_pose21_quant_set.py](tests/test_prepare_pose21_quant_set.py) | F | Q；选择器测试 | 导入业务模块、生成测试材料；本轮不运行，不视为新版本验收 |
| [prepare_pose21_fpga500.py](prepare_pose21_fpga500.py) | F | Q/T；候选 FPGA 子集 | 同目录选择器依赖、写集合；候选与最终 raw500 须区分 |
| [prepare_pose21_raw500.py](prepare_pose21_raw500.py) | F | T；样本选择与 R 通道提取 | 写图像/标签/清单；须来源及新目录授权 |
| [verify_pose21_raw500.py](verify_pose21_raw500.py) | F | T；原始集合独立核验 | 图像解码、全量读取；限定路径与资源 |
| [prepare_pose21_net1280.py](prepare_pose21_net1280.py) | F | T/C；网络输入转换 | grayprep/镜像绑定、写产物；须独立输出授权 |
| [run_pose21_stage4.py](run_pose21_stage4.py) | F | T；Docker 预处理/推理编排 | 固定 GPU、镜像及路径；须 Docker、GPU、资源和写入授权 |
| [eval_pose21_reference.py](eval_pose21_reference.py) | F | Q/T；Gray1 PT 匹配与误差 | 写 JSONL/CSV/标签；核对 device 与资源授权 |
| [eval_pose21_stage4.py](eval_pose21_stage4.py) | F | T；PT 评估/可视化 | 依赖 reference 模块、推理和写入；不覆盖已发布结果 |
| [verify_pose21_stage4.py](verify_pose21_stage4.py) | F | T；模型/输入/PT 产物审计 | 固定布局和哈希；须限定读取范围 |
| [inspect_pose21_onnx.py](inspect_pose21_onnx.py) | F | T；固定 weight-only ONNX 检查 | 顶层可解包并写模型，禁止当作无副作用模块导入；须写入授权 |
| [eval_pose21_onnx500.py](eval_pose21_onnx500.py) | F | T；同图 ONNX/PT 比较 | 依赖 stage4、固定 /dataset；核对 provider 和输出权限 |
| [verify_pose21_onnx500.py](verify_pose21_onnx500.py) | F | T；ONNX 记录与图像核验 | 固定 PT/ONNX 结果版本、全量读取；不是全测试集验收 |
| [prepare_pose21_cam2000.py](prepare_pose21_cam2000.py) | F | T/C；相机 PNG/BIN 生成 | grayprep 与样本绑定、写二进制；不得覆盖正式附件 |
| [export_single_cam2000.py](export_single_cam2000.py) | F | C；单图相机导出 | profile/grayprep 绑定；写 PNG/BIN，授权新输出 |
| [verify_pose21_cam2000.py](verify_pose21_cam2000.py) | F | T/C；独立解码审计 | 固定 /dataset、manifest、VERIFIED；限定读取，不等于 FPGA 通过 |
| [build_pose21_cam2000_index.py](build_pose21_cam2000_index.py) | F | T/C；样本类型索引 | 写固定 sample_index.csv；本轮禁止执行或改数据 |
| [verify_pose21_layout_v2.py](verify_pose21_layout_v2.py) | F | T；布局迁移身份核验 | 依赖 numbered-layout-v2 与历史包；非任意目录校验器 |
| [number_pose21_dataset_dirs.py](number_pose21_dataset_dirs.py) | O | T；初次目录编号 | 移动目录、写证据；禁止对已发布目录重跑 |
| [migrate_pose21_numbered_layout.py](migrate_pose21_numbered_layout.py) | O | T；编号引用迁移 | 修改路径/清单、生成历史包；不以整理为由重跑 |
| [backup_pose21_test500_nas.py](backup_pose21_test500_nas.py) | O | B/T；20261013 固定备份 | 固定 NAS 目标、复制与写清单；旧授权不继承 |
| [backup_h200_images_20261016.py](backup_h200_images_20261016.py) | O | B；10.16 镜像/源码备份 | Docker 导出、NAS 写入和空间占用；不代表最新覆盖 |
| [prepare_5090_env_branches.py](prepare_5090_env_branches.py) | O | G；历史迁移分支准备 | Git 分支/工作区变更；不重建已删分支，须新任务授权 |
| [sync_h200_main_only.py](sync_h200_main_only.py) | O | G；main-only 整理 | Git/工作区变更；不自动重跑或部署旧运行目录 |

## 执行与维护

先从 [RUNBOOK](../docs/agent/RUNBOOK.md)选[文档](../docs/agent/tasks/DOCUMENTATION.md)、[备份](../docs/agent/tasks/BACKUP.md)或[评估](../docs/agent/tasks/EVALUATION.md)卡，再读对应证据和实际参数。任务卡回答步骤与门禁；[模板统一入口](../docs/AGENT_GUIDE.md)回答如何描述任务。事实按[来源表](../docs/platform/SOURCES.md)维护。

归档候选的遍历没有充分的 junction/文件链接隔离保证；执行前必须审查源/目标链接和完整读取范围。冻结工具的同目录导入、绝对路径和原校验清单是保留原位的理由。若需适配新版本，另立任务，不改原实现。verify/inspect 文件名不保证无副作用；不批量导入、测试或执行。

包内脚本仍随原包保存；构建快照见 [task-artifacts](../task-artifacts/README.md)，不把其旧说明当作当前操作入口。

安全校验用法、验证范围和版本衔接见[最终交接](../docs/archive/2026-10-final-handoff/README.md)。只维护新入口的扫描/访问规则，旧实现不另行修复；不是两份并行维护的校验工具。
