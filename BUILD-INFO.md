# MoxServer 构建信息

- 包版本：`1.3.2`（来自主源码仓库 `MoxServer/package.yml`）
- 源码修订：`1ad62930749c576c76a21bb8e0ba800e163529c9`
- 构建时间（UTC）：`2026-10-11T16:02:16Z`
- 二进制构建：Go `1.26.5`、`CGO_ENABLED=0`、`-trimpath`、`-ldflags='-s -w'`
- 构建入口：主源码仓库 `scripts/github-build.sh`
- 平台：Linux、macOS、Windows，均包含 amd64 和 arm64
- 懒猫 LPK v2：`moxserver.lpk`，使用当前包配置及同批 Linux 二进制组装

源码修订用于追踪主源码仓库；本次从该修订的干净工作树重建全部平台二进制和 LPK，并同步包版本与部署文档。该修订不是本发布仓库的提交 SHA，不能用于 GitHub 文件下载地址。

- 本次更新：同步最新状态页、好友与群请求安全及保留策略修复、群 V2 投递和持久化优化。

全部发布文件的 SHA-256 摘要见 [SHA256SUMS](./SHA256SUMS)。
