# P0-5 Evidence: Permission Model

Capture date: `2026-04-23`  
App version: `Accio v0.7.1`

## User-facing result

The current desktop UI exposes two top-level execution modes, not four:

- `默认权限` — `Accio 自动在沙盒中运行命令`
- `完全访问权限` — `Accio 拥有对你的计算机的完全访问权限（高风险）`

Evidence:

- [p05-ui-permission-picker.png](/Users/a1-6/research/acciowork/06-screenshots/p05-ui-permission-picker.png)

## Actual runtime stack

The code and local policy files show a three-layer model:

| Layer | Values observed | Source |
|---|---|---|
| Policy decision | `allow`, `ask`, `deny` | `policy.jsonl`, `checkCommand(...)` |
| User approval outcome | `approved`, `approved_always`, `denied`, `denied_always`, `abort` | runtime enum in main bundle |
| Sandbox scope | `read_only`, `read_write_cwd`, `danger_full_access` | runtime sandbox enum |

Important clarification:

- I did **not** find a separate user-facing `restricted` enum in `policy.jsonl` or the current permission picker UI.
- The only `restricted` string I found in code is `WindowsRestrictedToken`, which is a Windows sandbox backend, not the macOS permission mode the user sees.

## Account-level policy schema

Observed schema in:

- `/Users/a1-6/.accio/accounts/1758713785/permissions/policy.jsonl`

Representative rows:

```json
{"type":"prefix","pattern":["write"],"decision":"ask","createdAt":"<REDACTED>"}
{"type":"prefix","pattern":["apply_patch"],"decision":"ask","createdAt":"<REDACTED>"}
{"type":"prefix","pattern":["sudo"],"decision":"deny","createdAt":"<REDACTED>"}
{"type":"prefix","pattern":["git","status"],"decision":"allow","bypassSandbox":true,"createdAt":"<REDACTED>"}
```

Observed account stats:

- `120` total rules
- decisions: `allow=104`, `ask=6`, `deny=10`
- `bypassSandbox=true` on `43` rules

## Agent-specific overrides

Observed agent policy files:

| Agent | Override summary |
|---|---|
| `Accio` | `write/edit/apply_patch = allow` |
| `Coder` | `write/edit/apply_patch = allow` |
| `国际站生意助手` | `write/edit/apply_patch = allow`, `browser = ask` |
| `Shopify Operator` | `write/edit = allow`, `apply_patch/bash/browser = ask` |
| `Ecommerce Mind` | `browser = ask` |

This means the product is not using one global tool policy only; each agent can narrow or widen the account default.

## Approval-state evidence

From the compiled main bundle:

```text
Wi.Approved = "approved"
Wi.ApprovedAlways = "approved_always"
Wi.Denied = "denied"
Wi.DeniedAlways = "denied_always"
Wi.Abort = "abort"
```

Trace taxonomy also distinguishes:

- `tool.permission.asked`
- `tool.permission.approved`
- `tool.permission.denied`
- `tool.permission.auto_granted`

Interpretation:

- `policy.jsonl` decides whether a tool auto-passes, asks, or auto-denies
- the approval UI and IM approval pipeline sit on top of that policy layer

## File-system and terminal scope

Main-bundle evidence shows:

- macOS sandbox backend is `seatbelt`
- execution shells through `/usr/bin/sandbox-exec`
- `danger_full_access` expands file-write policy to:
  - `(allow file-write* (regex #"^/"))`
- `read_write_cwd` derives writable roots from the working directory / configured roots
- some rules can additionally set `bypassSandbox=true`

The shell permission path therefore is **not** pure soft policy:

1. command prefix rules decide `allow / ask / deny`
2. a real macOS seatbelt sandbox is applied unless bypassed
3. `完全访问权限` maps to the unrestricted end of that sandbox layer

## Unmatched-command default

Critical code path:

```js
function Ah(e,t,a){return"allow"}
```

Combined with:

```js
const n = r.checkMultiple(s, e => "allow")
```

Interpretation:

- if no prefix rule matches and no heuristic escalates it, the default decision is `allow`
- safety on macOS then depends on sandbox scope unless the matched rule also carries `bypassSandbox=true`

## Bottom line

The permission model is best described as:

`policy engine` + `approval workflow` + `OS sandbox`

not as a single four-level switch.
