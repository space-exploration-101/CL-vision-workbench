# Agent Instructions — CL-vision-workbench

本文件面向自动化 Agent。不要把历史迁移文档当作当前状态。

## 必读顺序

1. 读取 [docs/agent/CONTEXT.md](docs/agent/CONTEXT.md)，获取最近记录的状态和专题入口；只展开本任务所需的主题。
2. 读取 [docs/agent/RUNBOOK.md](docs/agent/RUNBOOK.md)，只读取对应任务卡中的门禁、命令和验收。
3. 只有在对应场景下再读取：
   - 灾难恢复：`docs/reference/RECOVERY.md`
   - 环境重建：`rebuild/REBUILD_GUIDE.md`
   - 性能优化：`docs/reference/PERFORMANCE.md`
   - 迁移溯源：`docs/archive/README.md`

## 强制边界

- H200 使用 `ssh H200` 直连，禁止配置或使用 `wrs` 跳板。
- H200 是公用服务器。未经明确授权，不停止其他用户进程，不修改全局 Docker、驱动、内核、APT 或自动升级策略。
- GPU 使用前同时检查利用率、显存、进程所有者和持续采样；单次 `0%` 不等于空闲。
- 默认只做只读检查。启动量化、重启服务、挂载 NAS、写备份或删除数据前，确认任务授权覆盖该动作。
- 不在文件、命令参数、日志、Git 或回复中保存/复述明文密码。
- 系统备份明确排除 `data/output_data`。历史输出归档是独立任务，验证 NAS 副本后才允许清理本地。
- 禁止 `docker compose down -v`，禁止删除 APT/DPKG lock 文件，禁止用宽泛通配符递归删除共享盘数据。

## 临时材料隔离

- `work/` 仅用于存放临时提供给他人的材料，不属于项目源码、当前有效文档或运行状态的事实来源。
- 默认的仓库索引、全文检索、上下文收集、批量扫描和项目分析应排除 `work/`。只有在用户明确指定该目录中的路径或任务时才访问，并将处理范围限制在指定材料内。
- 不得根据 `work/` 中的内容判断或修改其他目录；处理其中的材料时，也不得让这些临时内容影响对项目代码、配置和正式文档的理解。

## 工作方式

- 开始时报告将检查什么、是否会使用 GPU、是否会修改状态。
- 先收集证据，再给判断；不要从历史文档推断当前 GPU、磁盘或服务状态。
- 变更采用可回滚步骤，大文件先写 `.partial`，校验后再原子改名。
- 每个任务结束时报告：实际变更、验证结果、未完成项和下一步。

## 文档分层

- 人类从 [docs/HANDOFF.md](docs/HANDOFF.md) 开始；Agent 不必把整套人类教程载入上下文。
- 文档维护先读 [任务卡](docs/agent/tasks/DOCUMENTATION.md)。当前状态保留一处详细来源，其他入口用摘要和链接，不追加多轮运行日志。
- `docs/archive/`、`rebuild/legacy/`、`.tmp/`、`task-artifacts/` 是历史或临时材料，只在具体任务需要时读取。
- 21类当前报告在 `docs/class-21/reports/current/`；地球、月球两类材料独立放在 [docs/Earth_Moon/](docs/Earth_Moon/README.md)。21类正式附件在 [outputs/class-21/](outputs/class-21/README.md)，过渡实验在 [历史实验归档](docs/archive/2026-09-class21-transition/README.md)。按报告中的精确路径访问附件，不批量展开；outputs 中的其他预览不作为验收依据。
- 本机根 Git 仓库只管理允许纳入的文档、原始数据和日志；`*.bin`、`*.zip` 暂不跟踪，work、临时脚本及依赖也不跟踪，不能用 `git add -f` 绕过规则。详见[版本管理](docs/VERSION_CONTROL.md)。未明确授权不提交、不设置远程或推送。
- 不用全局字符串替换更改 `quantitize-platform`、镜像、NAS、备份根目录或冻结历史路径。本机根目录现为 `E:\codex\CL-vision-workbench`；旧根路径只按历史证据解释。更名核验见[完成记录](docs/archive/2026-10-workbench-rename/RENAME_COMPLETION.md)。
- Git 不是完整环境备份。原始证据仍需独立副本与哈希；H200 最新源码目录和旧运行目录以[源码身份](docs/agent/topics/REPOSITORIES.md)区分。
