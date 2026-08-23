# MoxServer

[中文文档](./README.zh-CN.md)

MoxServer is the chat and event relay for MoxChat. It stores short-lived relay data for direct messages, group messages, friend requests, receipts, group metadata snapshots, user profiles, user data, and the unified event stream. It also participates in the Mox global mesh for discovery and cross-relay delivery.

## Release Files

Download the files from the release page that match your target platform:

| Target | File |
| --- | --- |
| Lazycat MicroServer | `moxserver.lpk` |
| Linux x64 | `moxserver-linux-amd64` |
| Linux arm64 | `moxserver-linux-arm64` |
| macOS Intel | `moxserver-darwin-amd64` |
| macOS Apple Silicon | `moxserver-darwin-arm64` |
| Windows x64 | `moxserver-windows-amd64.exe` |
| Windows arm64 | `moxserver-windows-arm64.exe` |

## Lazycat MicroServer Deployment

1. Download `moxserver.lpk`.
2. Install it from the Lazycat app UI, or with the CLI:

```sh
lzc-cli app install moxserver.lpk
```

3. Open the app at the assigned `moxserver` subdomain.
4. Check `https://<moxserver-host>/healthz`.

The LPK includes the MoxServer app process and a PostgreSQL service. Data is stored under Lazycat persistent storage, and the package health check uses `GET /healthz`.

## Linux Deployment

Create a PostgreSQL database owned by the runtime user:

```sql
CREATE USER moxserver WITH PASSWORD 'change-me';
CREATE DATABASE moxserver OWNER moxserver;
```

Run the binary:

```sh
chmod +x ./moxserver-linux-amd64
export MOXSERVER_ADDR=:8980
export MOXSERVER_DB_DSN='postgres://moxserver:change-me@127.0.0.1:5432/moxserver?sslmode=disable'
export MOXSERVER_DATA_RETENTION_DAYS=30
export MOXSERVER_MESH_PUBLIC_URL='https://relay.example.com'
./moxserver-linux-amd64
```

Use `moxserver-linux-arm64` on arm64 hosts.

## macOS Deployment

Use the matching macOS binary:

```sh
chmod +x ./moxserver-darwin-arm64
export MOXSERVER_ADDR=:8980
export MOXSERVER_DB_DSN='postgres://moxserver:change-me@127.0.0.1:5432/moxserver?sslmode=disable'
./moxserver-darwin-arm64
```

If macOS blocks a downloaded binary, remove the quarantine attribute:

```sh
xattr -d com.apple.quarantine ./moxserver-darwin-arm64
```

## Windows Deployment

Create the PostgreSQL database first, then start MoxServer from PowerShell:

```powershell
$env:MOXSERVER_ADDR = ":8980"
$env:MOXSERVER_DB_DSN = "postgres://moxserver:change-me@127.0.0.1:5432/moxserver?sslmode=disable"
$env:MOXSERVER_DATA_RETENTION_DAYS = "30"
$env:MOXSERVER_MESH_PUBLIC_URL = "https://relay.example.com"
.\moxserver-windows-amd64.exe
```

Use `moxserver-windows-arm64.exe` on Windows arm64 hosts.

## Environment Variables

This release directory includes a default `.env` file. MoxServer loads `.env` from the current directory or a parent directory; set `MOXSERVER_ENV_FILE` to use another file. Replace the database credentials and public relay URL before production use.

| Variable | Required | Description |
| --- | --- | --- |
| `MOXSERVER_ADDR` | No | Network address for the HTTP API, health check, and operations page. Default: `:8980`. |
| `MOXSERVER_DB_DSN` | Yes | PostgreSQL connection string used to store relay messages, events, profiles, groups, sessions, and schema state. |
| `MOXSERVER_ENV_FILE` | No | Alternate env file path. If omitted, the service searches for `.env` in the current directory and parent directories. |
| `MOXSERVER_DB_AUTO_INIT` | No | Set to `1` to create or ensure the application database and runtime user before connecting normally. |
| `MOXSERVER_DB_SUPER_DSN` | Auto-init only | PostgreSQL administrator DSN used only while `MOXSERVER_DB_AUTO_INIT=1`. |
| `MOXSERVER_DB_NAME` | Auto-init only | Database name that auto-init creates or verifies. |
| `MOXSERVER_DB_USER` | Auto-init only | Runtime PostgreSQL user that auto-init creates or verifies. |
| `MOXSERVER_DB_PASSWORD` | Auto-init only | Password assigned to the runtime PostgreSQL user during auto-init. |
| `MOXSERVER_DATA_RETENTION_DAYS` | No | Maximum retention window for relay data before background cleanup removes old records. Default: `30`. |
| `MOXSERVER_CHALLENGE_TTL_SECONDS` | No | Lifetime of challenge-response authentication codes. Default: `60`. |
| `MOXSERVER_AUTH_FAIL_WINDOW_MINUTES` | No | Time window used to count failed authentication attempts per source IP. Default: `30`. |
| `MOXSERVER_AUTH_FAIL_BAN_MINUTES` | No | Temporary ban duration after a source IP exceeds the failure threshold. Default: `30`. |
| `MOXSERVER_AUTH_FAIL_BAN_THRESHOLD` | No | Number of failed authentication attempts allowed in the window before banning the source IP. Default: `10`. |

## Mesh Network

Every MoxServer instance can act as a client relay, discovery provider, mailbox relay, group home/inbox relay, and bounded transit relay at the same time. The mesh keeps a persistent DiscoveryCatalog of verified relay advertisements while maintaining only a bounded neighbor set.

- Discovery starts from the built-in seeds `https://mox.ponzs.com` and `https://mesh.ponzs.com`, direct seed URLs, and remote JSON bootstrap documents.
- The catalog grows through bootstrap, authenticated gossip, DHT lookups, and anti-entropy synchronization. A failed endpoint is marked unhealthy for routing and retried later; it is not silently removed from the catalog.
- Relay identity comes from the signed `serverPub`; `nodeId` is derived from that identity. A URL is only an endpoint and cannot replace identity verification.
- Delivery resolves the exact destination first and tries an authenticated relay-to-relay connection directly. If direct delivery fails, a bounded multihop route may be used. Transit relays forward the envelope only; they do not write the destination mailbox, trigger push notifications, or retain another user's ciphertext.
- Unresolved delivery remains in the ingress durable outbox for retry. The mesh does not broadcast messages to unrelated relays.

The default neighbor limit is `32` and the default maximum multihop budget is `32` hops. DiscoveryCatalog size is independent of the neighbor limit.

### Mesh configuration

| Variable | Description |
| --- | --- |
| `MOXSERVER_MESH_ENABLED` | Enables the relay mesh. Default: `true`. |
| `MOXSERVER_MESH_DISCOVERY_ENABLED` | Enables bootstrap, catalog synchronization, gossip, and DHT discovery. Default: `true`. |
| `MOXSERVER_MESH_MULTIHOP_ENABLED` | Enables bounded relay-to-relay multihop fallback. Default: `true`. |
| `MOXSERVER_MESH_PUBLIC_URL` | Public origin advertised by this relay. Set the externally reachable HTTPS origin in production. |
| `MOXSERVER_MESH_SEED_RELAYS` | JSON array of complete signed relay advertisements supplied by an administrator. Each record is still verified before it enters the catalog. |
| `MOXSERVER_MESH_SEED_URLS` | JSON string array of direct relay URLs. If omitted, the two built-in seeds are used; `[]` disables built-in URL seeds. |
| `MOXSERVER_MESH_BOOTSTRAP_URLS` | JSON string array of remote JSON bootstrap document URLs. These URLs are read to obtain relay addresses, which are then verified and added to the local catalog. |
| `MOXSERVER_MESH_ALLOW_HTTP` | Allows `http://` mesh endpoints. Keep `false` for public deployments unless plain HTTP is intentional. |
| `MOXSERVER_MESH_ALLOW_PRIVATE_ENDPOINTS` | Allows private or reserved IP endpoints. Keep `false` on the public Internet; set `true` only for LAN/container deployments. |
| `MOXSERVER_MESH_MAX_PEERS` | Maximum resident mesh neighbors. Default: `32`. |
| `MOXSERVER_MESH_MAX_HOPS` | Maximum multihop forwarding budget. Default: `32`. |

Remote bootstrap URLs and direct seeds only inject verified relay advertisements into the discovery source list. They are not used as an alternate runtime configuration and do not replace addresses learned from gossip, DHT, or database state.

## Health and Operations

- `GET /healthz` verifies that the HTTP process is reachable.
- `POST /api/secure/health` verifies authenticated relay functionality and database access.
- `GET /` opens the operations/status page.
- `GET /status.json` returns a status snapshot.
- `GET /status/stream` streams live status updates.
- `GET /status/discovery.json` lists the current verified discovery catalog with stable pagination.
- `GET /api/mesh/v1/identity` returns the signed relay identity used by other relays to verify a seed.

Expose the service through HTTPS in production. MoxChat clients should use the externally reachable relay URL, for example `https://moxserver.example.com`.
