# 只读诊断

触发：已授权的服务、任务或资源只读排查。必要输入：目标主机、时间范围、服务或 task_id；只收集相关非敏感字段。诊断不授权重启、GPU 实验或修改配置。验收为状态、进程、资源和日志交叉对应，未复核项单列。

先读 [HOSTS](../topics/HOSTS.md)。以下 Linux 命令仅在 `ssh H200` 后执行。按任务选择检查，不要求本地任务执行服务器检查：

```bash
date -Is
hostname
cd /data3/ywang/quantitize-platform
git status --short
git rev-parse HEAD
docker compose -f deploy/docker-compose.yml ps
curl -fsS http://127.0.0.1:8000/health
curl -fsS http://127.0.0.1:8000/ready
curl -fsS http://127.0.0.1:8088/health
df -h /data1 /data3
nvidia-smi --query-gpu=index,uuid,memory.used,memory.free,utilization.gpu,power.draw --format=csv
nvidia-smi --query-compute-apps=gpu_uuid,pid,process_name,used_memory --format=csv
```

GPU 应间隔多次采样，并以 `ps -o user,pid,etime,stat,comm -p <pids>` 核对 owner 和时长。readiness 阈值不能替代任务峰值要求；只读取所需非敏感配置字段，不输出完整 `deploy/.env`。

任务诊断前确认 task_id 是合法单一目录名，再读取：

```bash
TASK_ID='<task_id>'
curl -fsS "http://127.0.0.1:8000/api/tasks/$TASK_ID" | python3 -m json.tool
pgrep -af '05_runner.py|quantitize.py|01_pt_eval.py|03_onnx_eval.py'
ls -lah "data/output_data/$TASK_ID/logs"
find "data/output_data/$TASK_ID" -type f -mmin -10 -print
```

- `running`：结合阶段进程、CPU/GPU 与文件变化；manifest 不变不等于卡住。
- `waiting_gpu`：结合 `/ready`、配置 UUID 和现场资源判断。
- `failed`：记录错误、最后完成阶段和日志，保留现场；诊断请求不自动授权修复。
- `completed`：核对该任务预期阶段全部完成、error 为空、ZIP 存在且可读。精度与实机结果分别验收。

旧 `fpga_eval` 与新 `fpga_input_roundtrip_eval` 必须查看实际脚本，不凭名称认定使用了 FPGA。


记录任务授权、时间、实际变更、身份与验证结果，未复核或未完成项明确标注。返回 [RUNBOOK](../RUNBOOK.md)。
