# 常用备份脚本快照

2026-11-10 按用户“放在同一路径下”的要求，从本机原位置逐字节收录四个程序。这里只维护带来源的冻结副本，原实现和[工具身份总索引](../../../tools/README.md)保持原位；不自动同步或修改两份代码。副本来源、大小和 SHA-256 见 [SOURCES.json](../SOURCES.json)。

| 本目录文件 | 原位置 | 实际能力与限制 |
|---|---|---|
| [archive_dataset_to_nas.py](archive_dataset_to_nas.py) | [operations/tools 原文件](../../../docs/class-21/operations/tools/archive_dataset_to_nas.py) | 生成 Dataset 分片和清单、抽样恢复后发布。虽然有参数，代码固定检查 136579 个文件和 897366345935 字节，并要求 NAS 根为空；不是可直接用于任意新数据集的通用备份器 |
| [verify_dataset_snapshot.py](verify_dataset_snapshot.py) | [operations/tools 原文件](../../../docs/class-21/operations/tools/verify_dataset_snapshot.py) | 核对共享、归档哈希、zstd、清单/catalog及原源目录 path/size/mtime；需要原源目录仍在，并扫描指定 NAS 根的 partial；不等于恢复后逐文件哈希验收 |
| [backup_pose21_test500_nas.py](backup_pose21_test500_nas.py) | [tools 原文件](../../../tools/backup_pose21_test500_nas.py) | 10-13 一次性复制/回读脚本，固定 H200 SOURCE 和 NAS TARGET；用于解释当时行为，不应直接重跑或改目标覆盖旧备份 |
| [backup_h200_images_20261016.py](backup_h200_images_20261016.py) | [tools 原文件](../../../tools/backup_h200_images_20261016.py) | 10-16 一次性补备预处理/API/Web并复核已有训练归档；固定路径和镜像身份，调用 Docker、写 NAS；不是数据解包脚本 |

## 使用前判断

这些程序主要用于 Linux/H200，需要 Python 3 及相应系统工具，如 GNU tar、zstd、sha256sum、findmnt；镜像任务另需 Docker 和 Git。Windows 上保存 `.py` 不意味着可以直接运行。不要通过执行或导入历史业务脚本来查询参数，先静态读文件。

归档程序会写 NAS；镜像程序还会导出镜像、采集依赖；校验程序会大量读取归档和源目录。运行前依据[备份任务卡](../../../docs/agent/tasks/BACKUP.md)核对本次范围、链接/路径、挂载、容量和授权。旧脚本并未证明具备全面链接隔离或适配新数据的能力。

本轮仅验证副本身份，没有运行程序或重新测试其业务行为。源目录已丢失时，按[恢复指南](../RESTORE.md)对展开内容执行 manifest 哈希验证；不要把旧快照校验器当作独立恢复工具。分片计数应明确检查实际文件数、snapshot 记录数、shards 记录数三者相等，不只依赖旧脚本成功文本。

若要适配新的数据集，另立有明确范围的变更任务，在原维护位置实现和验证；本目录另发布新日期快照并保留旧版本。脚本被仓库 `.gitignore` 排除，本机副本不会自动出现在 GitHub。
