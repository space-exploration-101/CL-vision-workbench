# 推荐的 Agent 工作流

## 组织方式

推荐使用“1 个协调 Agent + 最多 3 个专业 sub Agent”。协调 Agent 只维护工作板、接口决策和集成门禁；专业 Agent 每次只负责一个工作包。这样既能并行，又避免共享 H200 和两个 dirty worktree 上发生覆盖。

```text
Coordinator
├── Contract/Data Agent
├── Runtime/Training Agent
└── Evaluation/Quant/FPGA Agent
```

WP00、WP01 必须串行完成。WP01 schema 冻结后，WP02、WP03、WP04 可并行；WP05 之后再展开 WP06、WP07；WP08 需要硬件事实；WP09、WP10 最后集成。

## 每个 sub Agent 的领取规则

协调 Agent 给出的任务必须包含：

```yaml
work_package: WPxx
objective: 一个可验证的单一目标
scope_paths: 允许读取/修改的精确路径
read_first:
  - 根 AGENTS.md
  - docs/agent/CONTEXT.md
  - docs/agent/RUNBOOK.md
  - /data3/ywang/design/README.md
  - 对应 WP 文档
inputs: 固定 ID、manifest、commit、image digest
authorization: read-only | write-code | run-cpu | run-gpu | write-nas | deploy
non_goals: 本 Agent 明确不得做的工作
acceptance: 可执行命令和预期结果
handoff: 结果文档和状态更新位置
```

授权必须精确。`write-code` 不自动包含 GPU、部署、写 NAS、重启服务或删除数据。

## 当前授权档位（2026-09-12）

Coordinator 可在下列条件全部满足时，为 sub Agent 发放对应授权；未满足时保持只读并在工作板记录 blocker。

| 动作 | 可发放授权 | 强制门禁 |
|---|---|---|
| 在 `/data3/ywang` 创建目录/文件 | `write-code` 或任务专属 `write-data3` | 精确目标不存在；不得覆盖旧内容；owner/mode明确；`.partial` 校验后 no-replace 发布 |
| 使用 GPU | `run-gpu` | 持续采样利用率、显存、进程和所有者；单 GPU 显式绑定；预计与硬上限均小于 1 小时；只停止自己的任务 |
| 恢复镜像 | `restore-image` | manifest/digest通过；恢复目标为空；标签/镜像冲突 fail closed；禁止 prune、覆盖及修改 Docker daemon |
| 小规模训练 | `run-gpu` + `smoke-train` | 只验证 workflow；小 smoke 数据；单 GPU；不验精度；预计超过或无法保证小于 1 小时则不启动 |
| 重启服务 | `restart-project-service` | 仅 `ywang` 项目自有且精确点名的服务；记录owner/端口/依赖；具备健康检查和回滚；禁止系统/共享服务及其他用户进程 |

持续保留的禁止项：覆盖已有内容、写 NAS、未经另行授权在 `/data1` 创建目录、四卡或多卡任务、超过 1 小时的 GPU/训练任务、停止其他用户进程，以及修改全局 Docker、驱动、内核或 APT。一次任务获得 `run-gpu`、`restore-image` 或 `restart-project-service` 后，也不得把该权限传递给其他 WP 或扩大到未点名路径/服务。

GPU 空闲不能由单次 `0%` 推断。领取单必须记录候选 GPU ID、连续采样证据、显存余量、进程所有者、预计时长和硬超时；任一事实不明即不启动。

## 工作阶段

1. **Discover**：只读检查实时状态，保存 HEAD/status/diff、资源和依赖证据。
2. **Design**：先提交 schema/API/测试方案；跨仓库接口由协调 Agent 决策。
3. **Implement**：使用独立 branch/worktree；一次提交只解决一个假设或契约变化。
4. **Verify**：先 unit/golden，再 CPU smoke，再按授权执行 GPU/HIL；失败证据保留。
5. **Handoff**：报告实际变更、验证结果、未完成项、风险、commit 和 artifact hash。
6. **Integrate**：协调 Agent 运行跨仓库 contract tests，决定合并和下一个工作包。

## 共享工作区纪律

- 当前 H200 两仓库有未提交修改。第一位 Agent 不得基于自己的判断提交、丢弃或整理它们；先由所有者确认归属并固化。
- 并行 Agent 不直接共享同一 working tree；使用不同 worktree、branch、task output 和 scratch 目录。
- schema、公共 profile、manifest 字段由单一 owner 修改，其他 Agent 通过评审后的版本消费。
- GPU/HIL 同一时间只有一个任务 owner；未获得资源授权的 Agent 只写计划和 CPU 测试。
- 不让 sub Agent 清理其他 Agent 的 scratch。每个任务持有显式路径清单，清理需在 handoff 中报告。

## 推荐工作板

协调 Agent 维护 `status/WORK_BOARD.md`，字段如下：

| WP | owner | state | branch/worktree | inputs frozen | last evidence | blocker | next gate |
|---|---|---|---|---|---|---|---|

状态只使用：`BACKLOG`、`DISCOVERY`、`DESIGN`、`IMPLEMENTING`、`VERIFYING`、`BLOCKED`、`DONE`。`DONE` 必须满足对应 WP 的验收条款。

## sub Agent 最终报告模板

```yaml
work_package:
status: DONE|BLOCKED|FAILED
observed_at:
inputs:
  git_commits: []
  image_digests: []
  manifest_hashes: []
changes: []
verification: []
artifacts: []
risks: []
remaining: []
recommended_next_gate:
```

## 不推荐的编排方式

- 多个 Agent 同时修改 manifest/schema 或同一个 runner。
- 在未冻结数据和镜像时比较性能或精度。
- 将“代码已写完”等同于工作包完成。
- 让 Agent 根据单次 `nvidia-smi 0%` 判断 GPU 可用。
- 让实施 Agent自行放宽验收阈值以使结果通过。
