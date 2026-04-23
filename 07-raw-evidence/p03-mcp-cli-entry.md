# P0-3 Evidence: `accio-mcp.mjs` Real Role

Capture date: `2026-04-23`  
App version: `Accio v0.7.1`

## CLI file

- Bundle path:
  [accio-mcp.mjs](/Applications/Accio.app/Contents/Resources/accio-mcp-cli/accio-mcp.mjs:1)

## What the CLI actually does

Observed commands from the entry file and `--help`:

- `toolkit`
- `search`
- `call`
- `server list`
- `server add`
- `server remove`
- `server test`
- `server auth`

Observed base URLs in the file:

- `http://127.0.0.1:${GATEWAY_PORT||4097}/mcp/proxy`
- `http://127.0.0.1:${GATEWAY_PORT||4097}/mcp/oauth`
- `http://127.0.0.1:${GATEWAY_PORT||4097}/mcp/custom`

Observed env wiring:

- `ACCIO_TRACE_CONTEXT`
- `ACCIO_MCP_JSON`

Conclusion:

- this file is a local CLI client to the desktop app’s gateway
- it is not itself an MCP server
- it does not expose a separate public TCP service beyond the app’s local gateway

## Main-process coupling

Relevant main-bundle evidence from:
[p02-out-main-index.js](/Users/a1-6/research/acciowork/07-raw-evidence/p02-out-main-index.js)

Observed routes and strings:

- `/mcp/proxy`
- `/mcp/oauth`
- `/mcp/custom`
- `/mcp/tools`
- loopback-only `Forbidden` branch
- shell runner support for `ACCIO_MCP_JSON`
- trace injection for `ACCIO_TRACE_CONTEXT`

This shows the main process:

1. owns the local MCP gateway
2. augments CLI calls with tracing / JSON env injection
3. guards the local service from non-loopback callers

## Live port check

`lsof -nP -iTCP:4097 -sTCP:LISTEN`

```text
COMMAND  PID USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
Accio   7714 a1-6   68u  IPv6 0x94bf25c2f8225b7d      0t0  TCP *:4097 (LISTEN)
```

## Health check: loopback vs LAN

Local loopback request:

```http
GET http://127.0.0.1:4097/health

HTTP/1.1 200 OK
content-type: application/json

{"healthy":true,"platform":"darwin","uptime":37097.752840375,"bootPhase":"app_ready","bootTiming":{"authReadyAt":1776921789581,"appReadyAt":1776921789608,"phaseEnteredAt":1776921789608}}
```

LAN IP request from the same machine:

```http
GET http://192.168.0.137:4097/health

HTTP/1.1 403 Forbidden
content-type: text/plain; charset=UTF-8

Forbidden
```

Conclusion:

- the process listens on `*:4097`
- access control is enforced in app logic, not by binding only to `127.0.0.1`
- on this machine, the gateway is externally bound but practically loopback-only
