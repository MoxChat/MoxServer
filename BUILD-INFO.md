# MoxServer 构建信息

| 项目 | 内容 |
| --- | --- |
| 包版本 | `1.3.2` |
| 服务源码修订 | [f47914f5](https://github.com/ponzS/MoxServer/commit/f47914f55eb45ce15d59f78f9144bd5b86dc2643) |
| 构建开始时间（UTC） | `2026-11-22T10:56:17Z` |
| 工作区构建入口 | [scripts/github-build.sh](https://github.com/ponzS/MoxWorkSpace/blob/1c04a18c10549910f3d7711038cca9ce04f75c83/scripts/github-build.sh) |
| 工具链 | Go `1.26.5`、LazyCat CLI `2.0.9` |
| 二进制构建参数 | `CGO_ENABLED=0`、`-trimpath`、`-ldflags='-s -w'` |
| 目标平台 | Linux、macOS、Windows，均提供 amd64 和 arm64 |
| 懒猫 LPK v2 | `moxserver.lpk`，内含同批次的 Linux amd64 和 arm64 二进制 |

更新签名设备会话和群 V2 投递的授权检查。

为固定下载版本，请使用本发布仓库的提交 SHA。服务源码修订用于追踪构建来源。校验摘要见 [SHA256SUMS](./SHA256SUMS)。
