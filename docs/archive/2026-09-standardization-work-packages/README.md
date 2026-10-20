# 2026-09 标准化工作包本地材料归档

> 归档日期：2026-09-18  
> 状态：历史设计和执行证据，不是当前授权、工作板或代码入口

## 1. 归档来源

本目录收纳项目根目录中 2026-09-03 至 2026-09-16 形成的临时材料：

```text
.auth-README.md
.auth-11_AGENT_WORKFLOW.md
.wp01-WORK_BOARD.md
wp04_design_tmp/WORK_BOARD.md
wp08a_impl_tmp/
job_config_class21_tmp.json
```

这些文件用于早期跨仓库标准化设计、Agent 工作包编排，以及在本机临时审阅 H200 worktree 内容。它们不由当前程序读取，项目内也没有有效引用。

## 2. 保留内容

| 归档文件 | 含义 |
|---|---|
| `design/README_20260912.md` | 2026-09-12 全流程标准化设计和当时的授权边界 |
| `design/AGENT_WORKFLOW_20260912.md` | 当时建议的 Coordinator/WP 多 Agent 工作方式 |
| `status/WORK_BOARD_20260912_1422.md` | WP01 阶段的早期工作板快照 |
| `status/WORK_BOARD_20260916_2243.md` | WP04/WP08-A 等后续状态的工作板快照 |
| `legacy/20260903_class21_r_only_job_config.json` | 早期 21 类三通道 R-only、400/84 数据方案的任务配置 |
| `wp08a/pc_test_package.py` | WP08-A PC-only 测试包实现快照 |
| `wp08a/test_pc_test_package.py` | 对应的 CPU 契约测试快照 |
| `wp08a/WP08_A_PC_TEST_PACKAGE.md` | 对应设计和使用边界 |

WP08-A 三个文件在归档前与 H200 worktree `/data3/ywang/worktrees/wp08a-pc-test-package` 的 commit `c922428f6e8c7d5b9572a86e31cf30a7da96e799` 核对一致；H200 worktree 当时为 clean。保留本地副本是因为该分支记录显示尚未 push/merge。

## 3. 未保留内容

`wp08a_impl_tmp` 中的 `bundle_reference.py`、`outcome_reference.py`、`trace_reference.py` 及对应测试只是为了脱离原 worktree 阅读或试跑而复制的依赖，不是 WP08-A 的正式新增文件。`__pycache__/*.pyc` 是 Python 自动生成缓存。上述内容已删除。

## 4. 当前使用边界

- 当前操作授权只以用户当前请求、根目录 `AGENTS.md`、`docs/agent/CONTEXT.md` 和 `docs/agent/RUNBOOK.md` 为准；本归档中的旧授权文字不自动延续。
- 当前工作包状态应从 H200 `/data3/ywang/design/status/WORK_BOARD.md` 重新读取，不能从两份历史快照推断。
- `20260903_class21_r_only_job_config.json` 属于三通道 R-only 过渡模型，不适用于当前 `[B,1,1280,1280]` Gray1 模型。
- WP08-A 快照仅代表 L0 CPU 合同实现，固定为 software-only、non-promotable；不代表真实量化、FPGA/HIL、生产或发布已经通过。
- 若继续开发 WP08-A，应从对应 Git commit/worktree 恢复和集成，不应直接把归档文件复制回当前代码树。
