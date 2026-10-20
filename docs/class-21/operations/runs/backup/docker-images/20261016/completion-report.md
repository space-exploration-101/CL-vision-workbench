# H200 当前 Docker 镜像 NAS 归档完成记录

完成时间：2026-10-16 14:03 CST。用户授权整理已有训练/PT 备份，并在同一共享根补备 data-pipeline、quantitize 镜像。本轮不删除镜像、不使用 GPU、不重启生产服务。

## 实际变更

NAS 根为 `\\10.2.26.26\902_data\0-项目\13-专项\4-代码\20261016_H200_docker_images`，已确认来源 CIFS `//10.2.26.26/902_data`。已有训练目录已按 records-v1 整理，因此保留其路径及全部文件，重新验证 22 项清单。新增目录为 `yolo_gray1_data_pipeline`、`quantitize_platform_api`、`quantitize_platform_web`。

新增资产目录均包含完整未压缩 Docker save TAR、固定 HEAD 的 source.bundle/source.tar.gz、Dockerfile、README、SHA256SUMS、VERIFIED，身份、运行依赖版本与校验证据置于 records/。根新增 README、INDEX.json、SHA256SUMS、VERIFIED 和集合级 records/backup-20261016.json。

| 镜像 | ID 前缀 | TAR 字节数 | TAR SHA-256 |
|---|---|---:|---|
| ywang/yolo-gray1-data-pipeline:0.2.1 | ae238a8a9099 | 361152512 | 7baa5c1050ee25768b182383453da581194f4f2ffdbc830aff56f0d23a69c011 |
| quantitize-platform-api:latest | f633b01c2f85 | 11658015232 | 76ce1e923dc8ff12584472cfab47dd8174438c98081d6091cecaad6d550b13ed |
| quantitize-platform-web:latest | 7aef48747d3f | 191891968 | 74c7a66c2f255668e22f676f5e101f55f07d3c5f2dbf9741adab51d3e0fc167e |

预处理源码提交 f147cee93c83876348205ca268eb930010192116；平台源码提交 b1ac5ea84e0374d9bfa2be2c9214e07e950a0a79。平台宿主源码提交晚于镜像构建，分别记录，不将其声称为镜像原始构建提交。本轮未推送 GitHub、未打包未知未跟踪文件、凭据、模型、数据集、scratch 或 output_data。

## 验证结果

新增每个目录 9 项文件清单校验通过；镜像 TAR 配置哈希等于完整 image ID，全部层 diff ID 通过，NAS 原子改名后独立回读 SHA-256 通过，Git bundle 验证通过。根清单覆盖总索引、集合记录和各资产清单/标记；最终检查无 .partial 或 Docker 临时文件。

无网络、只读、限额 CPU 临时容器收集 Python/系统已安装包版本，未使用 GPU。API/Web 容器 ID、镜像 ID 和启动时间备份前后相同；health 前后均 HTTP 200，API ready 前后均 HTTP 503。未修复该既有异常。

## 未完成项与下一步

资产状态为 ASSET_VERIFIED，整套平台保持 PARTIAL。新增三个归档未执行 docker load、独立 Docker daemon 恢复或业务样本验证。完整镜像包含已安装运行依赖，版本记录不是可安装的 wheel/deb 包仓库；API Dockerfile 缺失的基础镜像标签也未修复。后续优先在独立环境导入、恢复宿主挂载源码与配置并做 CPU 固定样本验证，再补离线构建依赖。数据、模型和 FPGA 发布验收属于独立范围。

恢复入口见 [NAS 备份与环境恢复](../../../../../knowledge/04_NAS备份与环境恢复.md#34-2026-10-16-预处理apiweb-镜像与总索引)。本次执行脚本为仓库根 `tools/backup_h200_images_20261016.py`；它是一次性创建脚本，目标存在完整标记时复核，未完成目录需先核对现场，不能直接覆盖重跑。
