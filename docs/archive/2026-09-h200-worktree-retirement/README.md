# H200 旧本机镜像退役归档

2026-09-18 删除 `E:\codex\quantitize\h200-worktree` 前，从该独立 Git 仓库中保留了未纳入 Git 且没有完整外部副本的文件。

- `raw/.env.gpu3`：当时的本地 GPU 和演示模式配置，不得当作当前配置使用。
- `raw/docs/archive/README.md`：旧镜像内的归档入口。
- `raw/docs/archive/project-root-legacy/`：旧项目根目录的迁移、恢复和设备记录。
- `SHA256SUMS`：删除前源文件与归档副本共同的 SHA-256。

该镜像的 Git 工作区在退役前为干净状态，HEAD 为 `5632157eabcb57ad9f6ee50032d01e8d692d285a`；GitHub 远端 `main` 已包含并继续向前推进。H200 上的 `/data3/ywang/quantitize-platform` 是独立仓库，不依赖该本地镜像。
