# MoxServer

[English](./README.md)

MoxServer 是 MoxChat 的聊天与事件中继，负责私聊消息、群聊消息、好友申请、回执、群资料快照、用户资料、用户数据和统一事件流，同时参与 Mox 全球网状网络中的发现和跨中继投递。

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
export MOXSERVER_MESH_PUBLIC_URL='https://relay.example.com'
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
$env:MOXSERVER_MESH_PUBLIC_URL = "https://relay.example.com"
.\moxserver-windows-amd64.exe
```

Windows arm64 主机使用 `moxserver-windows-arm64.exe`。

## 环境变量

发布目录中已经包含默认 `.env` 文件。MoxServer 会从当前目录或上级目录自动加载 `.env`；如需使用其他文件，可设置 `MOXSERVER_ENV_FILE`。生产环境请替换数据库密码和公网中继地址。

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

## 网状网络

每个 MoxServer 实例都可以同时承担客户端接入、发现提供者、用户邮箱中继、群归属/收件中继和有界中转中继角色。网状网络会持久保存经过验证的 DiscoveryCatalog，同时只维护数量有界的常驻邻居连接。

- 自动发现默认从内置 `https://mox.ponzs.com` 和 `https://mesh.ponzs.com`、直接 seed URL 以及远程 JSON bootstrap 文档开始。
- 目录会通过 bootstrap、认证 Gossip、DHT 查询和反熵同步持续增长。失效地址只会在路由健康视图中降级并后台重试，不会从发现目录中静默删除。
- 中继身份来自签名的 `serverPub`，`nodeId` 由该身份派生。URL 只是 endpoint，不能替代身份校验。
- 投递时先精确解析目标并优先尝试经过认证的中继直连；直连失败后才使用有界多跳。中间中继只转发信封，不写入目标邮箱、不触发推送，也不长期保存其他用户的密文。
- 暂时无法到达的消息会保留在入口中继的 durable outbox 中重试，不会广播给无关中继。

默认常驻邻居上限为 `32`，默认最大多跳预算为 `32` 跳。发现目录记录数量与邻居连接上限相互独立。

### 网状网络配置

| 变量 | 说明 |
| --- | --- |
| `MOXSERVER_MESH_ENABLED` | 启用中继网状能力，默认 `true`。 |
| `MOXSERVER_MESH_DISCOVERY_ENABLED` | 启用 bootstrap、目录同步、Gossip 和 DHT 发现，默认 `true`。 |
| `MOXSERVER_MESH_MULTIHOP_ENABLED` | 启用有界中继多跳 fallback，默认 `true`。 |
| `MOXSERVER_MESH_PUBLIC_URL` | 本中继对外公布的 origin。生产环境应设置为外部可访问的 HTTPS 地址。 |
| `MOXSERVER_MESH_SEED_RELAYS` | 管理员提供的完整签名中继广告 JSON 数组。每条记录仍需验证通过后才能进入目录。 |
| `MOXSERVER_MESH_SEED_URLS` | 直接中继 URL 的 JSON 字符串数组。不配置时使用两个内置地址，设为 `[]` 可关闭内置 URL seed。 |
| `MOXSERVER_MESH_BOOTSTRAP_URLS` | 远程 JSON bootstrap 文档 URL 的 JSON 字符串数组。服务端读取文档中的地址，验证后注入本地发现目录。 |
| `MOXSERVER_MESH_ALLOW_HTTP` | 是否允许 `http://` Mesh endpoint。公网部署建议保持 `false`。 |
| `MOXSERVER_MESH_ALLOW_PRIVATE_ENDPOINTS` | 是否允许私网或保留 IP endpoint。公网部署保持 `false`，局域网或容器组网才设为 `true`。 |
| `MOXSERVER_MESH_MAX_PEERS` | 常驻 Mesh 邻居最大数量，默认 `32`。 |
| `MOXSERVER_MESH_MAX_HOPS` | 多跳转发最大跳数，默认 `32`。 |

远程 bootstrap URL 和直接 seed 只负责把验证通过的中继广告注入当前发现源列表，不会替代运行时配置，也不会覆盖 Gossip、DHT 或数据库中学习到的地址。

## 健康检查和运维

- `GET /healthz` 检查 HTTP 进程是否可达。
- `POST /api/secure/health` 检查已认证的中继功能和数据库访问。
- `GET /` 打开运维状态页。
- `GET /status.json` 返回状态快照。
- `GET /status/stream` 返回实时状态流。
- `GET /status/discovery.json` 返回当前已验证发现目录，支持稳定分页。
- `GET /api/mesh/v1/identity` 返回供其他中继验证 seed 使用的签名中继身份。

生产环境建议通过 HTTPS 暴露服务。MoxChat 客户端应填写外部可访问的中继地址，例如 `https://moxserver.example.com`。
