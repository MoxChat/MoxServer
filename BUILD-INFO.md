# MoxServer 构建信息

- 包版本：`1.3.0`（来自主源码仓库 `MoxServer/package.yml`）
- 源码修订：`d5d69f63d632d3fc6b7ef86ffa4613412f62233f`
- 构建时间（UTC）：`2026-10-04T11:45:25Z`
- 二进制构建：Go `1.26.5`、`CGO_ENABLED=0`、`-trimpath`、`-ldflags='-s -w'`
- 构建入口：主源码仓库 `scripts/github-build.sh`
- 平台：Linux、macOS、Windows，均包含 amd64 和 arm64
- 懒猫 LPK v2：`moxserver.lpk`，使用当前包配置及同批 Linux 二进制组装

源码修订用于追踪主源码仓库；本次仅调整发布构建流程与文档，服务源码无未提交修改。该修订不是本发布仓库的提交 SHA，不能用于 GitHub 文件下载地址。

全部发布文件的 SHA-256 摘要见 [SHA256SUMS](./SHA256SUMS)。
