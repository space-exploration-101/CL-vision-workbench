# 工具使用边界

继承根 [AGENTS](../AGENTS.md)。先读 [README](README.md) 对照任务与证据；脚本含写入、备份和 Git 分支操作，不批量执行，不因文件名含 verify 就假定完全只读。

文档校验只运行 check_documentation_safe.py，工具测试只指定 tests/test_check_documentation_safe.py，不批量发现业务测试。原 check_documentation.py 保留旧实现与解析辅助，勿直接执行。其他脚本按对应报告固定模型、路径、资源和授权，不因文档维护修改算法或运维脚本。

逐项身份只在 README 维护；按[文档任务卡](../docs/agent/tasks/DOCUMENTATION.md)检查边界后执行本地校验，严格隔离须限定文件集并拒绝重解析点。不得以检查参数为由导入或执行冻结脚本。
