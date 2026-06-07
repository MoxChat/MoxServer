# MoxServer

[中文文档](./README.zh-CN.md)

MoxServer is the chat and event relay for MoxChat. It stores short-lived relay data for direct messages, group messages, friend requests, receipts, group metadata snapshots, user profiles, user data, and the unified event stream.

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
.\moxserver-windows-amd64.exe
```

Use `moxserver-windows-arm64.exe` on Windows arm64 hosts.

## Environment Variables

This release directory includes a default `.env` file. When the binary is started from this directory, MoxServer loads `.env` automatically; set `MOXSERVER_ENV_FILE` to point at a different file. The default file is a deployment template, so replace `change-me` database credentials before production use.

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

## Health and Operations

- `GET /healthz` verifies that the HTTP process is reachable.
- `POST /api/secure/health` verifies authenticated relay functionality and database access.
- `GET /` opens the operations/status page.
- `GET /status.json` returns a status snapshot.
- `GET /status/stream` streams live status updates.

Expose the service through HTTPS in production. MoxChat clients should use the externally reachable relay URL, for example `https://moxserver.example.com`.
