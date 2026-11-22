# MoxServer 构建信息

| 项目 | 内容 |
| --- | --- |
| 包版本 | `1.3.3` |
| 服务源码修订 | [c81102d8](https://github.com/ponzS/MoxServer/commit/c81102d866ac6dee0c557b31fd6c1fef975c28e8) |
| 构建开始时间（UTC） | `2026-11-22T12:00:09Z` |
| 工作区构建入口 | [scripts/github-build.sh](https://github.com/ponzS/MoxWorkSpace/blob/1c04a18c10549910f3d7711038cca9ce04f75c83/scripts/github-build.sh) |
| 工具链 | Go `1.26.5`、LazyCat CLI `2.0.9` |
| 二进制构建参数 | `CGO_ENABLED=0`、`-trimpath`、`-ldflags='-s -w'` |
| 目标平台 | Linux、macOS、Windows，均提供 amd64 和 arm64 |
| 懒猫 LPK v2 | `moxserver.lpk`，内含同批次的 Linux amd64 和 arm64 二进制 |

1.3.3 修复从两种已发布的数据库迁移版本 15 升级时的启动失败。服务启动后会保留原有迁移记录并新增版本 16。

为固定下载版本，请使用本发布仓库的提交 SHA。服务源码修订用于追踪构建来源。校验摘要见 [SHA256SUMS](./SHA256SUMS)。
