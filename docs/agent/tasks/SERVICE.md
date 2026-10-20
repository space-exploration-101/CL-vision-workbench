# 服务启动与重启

必要输入：授权覆盖的服务、允许中断范围、当前任务清单及诊断结果。只读取[运行环境](../../platform/02_H200训练测试推理与量化运行环境.md)与现场配置中需要的部分；完整主机恢复转 [rebuild](../../../rebuild/README.md)，不在本卡扩展构建教程。

仅适用于已授权的服务操作，先按 [DIAGNOSTICS](DIAGNOSTICS.md) 检查现场。先查询任务列表、相关任务详情和 runner/worker 进程。存在运行或等待任务时，仅在授权明确包含中断这些任务后继续。

`up -d --no-build` 用于启动或按配置更新，不会重启配置未变的已有容器。授权重启现有服务后执行：

```bash
cd /data3/ywang/quantitize-platform
docker compose -f deploy/docker-compose.yml config --services
docker compose -f deploy/docker-compose.yml restart <已核对的API服务名> <已核对的Web服务名>
docker compose -f deploy/docker-compose.yml ps
curl -fsS http://127.0.0.1:8000/health
curl -fsS http://127.0.0.1:8000/ready
curl -fsS http://127.0.0.1:8088/health
```

用现场服务名替换占位符，确认历史任务仍可查询。`/ready` 不通过需说明是否共享 GPU 忙；不能以 health 成功替代资源就绪。禁止 `down -v`；镜像重建、GPU 配置变更和强制重建容器需在授权范围内。


记录任务授权、时间、实际变更、身份与验证结果，未复核或未完成项明确标注。返回 [RUNBOOK](../RUNBOOK.md)。
