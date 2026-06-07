# MoxServer

[English](./README.md)

MoxServer 是 MoxChat 的聊天与事件中继，负责私聊消息、群聊消息、好友申请、回执、群资料快照、用户资料、用户数据和统一事件流。

## 发布文件

从发布页下载与你的部署目标匹配的文件：

| 目标 | 文件 |
| --- | --- |
| 懒猫微服 | `moxserver.lpk` |
| Linux x64 | `moxserver-linux-amd64` |
| Linux arm64 | `moxserver-linux-arm64` |
| macOS Intel | `moxserver-darwin-amd64` |
| macOS Apple Silicon | `moxserver-darwin-arm64` |
| Windows x64 | `moxserver-windows-amd64.exe` |
| Windows arm64 | `moxserver-windows-arm64.exe` |

## 懒猫微服部署

1. 下载 `moxserver.lpk`。
2. 在懒猫应用界面安装，或使用 CLI：

```sh
lzc-cli app install moxserver.lpk
```

3. 打开分配到的 `moxserver` 子域名。
4. 检查 `https://<moxserver-host>/healthz`。

LPK 内包含 MoxServer 进程和 PostgreSQL 服务。数据保存在懒猫持久化目录中，包内健康检查使用 `GET /healthz`。

## Linux 部署

先创建 PostgreSQL 数据库：

```sql
CREATE USER moxserver WITH PASSWORD 'change-me';
CREATE DATABASE moxserver OWNER moxserver;
```

启动二进制：

```sh
chmod +x ./moxserver-linux-amd64
export MOXSERVER_ADDR=:8980
export MOXSERVER_DB_DSN='postgres://moxserver:change-me@127.0.0.1:5432/moxserver?sslmode=disable'
export MOXSERVER_DATA_RETENTION_DAYS=30
./moxserver-linux-amd64
```

arm64 主机使用 `moxserver-linux-arm64`。

## macOS 部署

使用匹配的 macOS 二进制：

```sh
chmod +x ./moxserver-darwin-arm64
export MOXSERVER_ADDR=:8980
export MOXSERVER_DB_DSN='postgres://moxserver:change-me@127.0.0.1:5432/moxserver?sslmode=disable'
./moxserver-darwin-arm64
```

如果 macOS 拦截下载的二进制，移除 quarantine 属性：

```sh
xattr -d com.apple.quarantine ./moxserver-darwin-arm64
```

## Windows 部署

先创建 PostgreSQL 数据库，然后在 PowerShell 中启动：

```powershell
$env:MOXSERVER_ADDR = ":8980"
$env:MOXSERVER_DB_DSN = "postgres://moxserver:change-me@127.0.0.1:5432/moxserver?sslmode=disable"
$env:MOXSERVER_DATA_RETENTION_DAYS = "30"
.\moxserver-windows-amd64.exe
```

Windows arm64 主机使用 `moxserver-windows-arm64.exe`。

## 环境变量

发布目录中已经包含默认 `.env` 文件。从该目录启动二进制时，MoxServer 会自动加载 `.env`；如需使用其他文件，可设置 `MOXSERVER_ENV_FILE`。默认 `.env` 是部署模板，生产环境必须先替换 `change-me` 数据库密码等占位值。

| 变量 | 必填 | 说明 |
| --- | --- | --- |
| `MOXSERVER_ADDR` | 否 | HTTP API、健康检查和运维状态页监听地址，默认 `:8980`。 |
| `MOXSERVER_DB_DSN` | 是 | PostgreSQL 连接串，用于存储中继消息、事件、用户资料、群信息、会话和表结构状态。 |
| `MOXSERVER_ENV_FILE` | 否 | 指定其他 env 文件路径；不填时服务会在当前目录或上级目录查找 `.env`。 |
| `MOXSERVER_DB_AUTO_INIT` | 否 | 设为 `1` 时，在正常连接前自动创建或确认业务数据库和运行用户。 |
| `MOXSERVER_DB_SUPER_DSN` | 仅自动初始化 | 仅在 `MOXSERVER_DB_AUTO_INIT=1` 时使用的 PostgreSQL 管理员连接串。 |
| `MOXSERVER_DB_NAME` | 仅自动初始化 | 自动初始化时创建或确认存在的数据库名。 |
| `MOXSERVER_DB_USER` | 仅自动初始化 | 自动初始化时创建或确认存在的运行期数据库用户。 |
| `MOXSERVER_DB_PASSWORD` | 仅自动初始化 | 自动初始化时写入运行期数据库用户的密码。 |
| `MOXSERVER_DATA_RETENTION_DAYS` | 否 | 中继数据最长保留窗口，后台清理会删除超出窗口的旧记录，默认 `30`。 |
| `MOXSERVER_CHALLENGE_TTL_SECONDS` | 否 | challenge-response 认证验证码有效期，默认 `60` 秒。 |
| `MOXSERVER_AUTH_FAIL_WINDOW_MINUTES` | 否 | 按源 IP 统计认证失败次数的时间窗口，默认 `30` 分钟。 |
| `MOXSERVER_AUTH_FAIL_BAN_MINUTES` | 否 | 源 IP 超过失败阈值后的临时封禁时长，默认 `30` 分钟。 |
| `MOXSERVER_AUTH_FAIL_BAN_THRESHOLD` | 否 | 统计窗口内允许的认证失败次数，超过后封禁该源 IP，默认 `10`。 |

## 健康检查和运维

- `GET /healthz` 检查 HTTP 进程是否可达。
- `POST /api/secure/health` 检查已认证的中继功能和数据库访问。
- `GET /` 打开运维状态页。
- `GET /status.json` 返回状态快照。
- `GET /status/stream` 返回实时状态流。

生产环境建议通过 HTTPS 暴露服务。MoxChat 客户端应填写外部可访问的中继地址，例如 `https://moxserver.example.com`。
