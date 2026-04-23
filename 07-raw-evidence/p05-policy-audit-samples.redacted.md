# P0-5 Evidence: Policy and Audit Samples

Capture date: `2026-04-23`  
App version: `Accio v0.7.1`

## Account policy shape

Source:

- `/Users/a1-6/.accio/accounts/1758713785/permissions/policy.jsonl`

JSONL row shape observed:

```json
{
  "type": "prefix",
  "pattern": ["write"],
  "decision": "ask",
  "justification": "optional",
  "bypassSandbox": true,
  "createdAt": "<REDACTED>"
}
```

Account stats:

- rows: `120`
- decisions: `allow=104`, `ask=6`, `deny=10`
- `bypassSandbox=true`: `43`

Ask rules observed:

```json
{"type":"prefix","pattern":["git","reset"],"decision":"ask","createdAt":"<REDACTED>"}
{"type":"prefix","pattern":["git","rm"],"decision":"ask","createdAt":"<REDACTED>"}
{"type":"prefix","pattern":["rm"],"decision":"ask","createdAt":"<REDACTED>"}
{"type":"prefix","pattern":["write"],"decision":"ask","createdAt":"<REDACTED>"}
{"type":"prefix","pattern":["edit"],"decision":"ask","createdAt":"<REDACTED>"}
{"type":"prefix","pattern":["apply_patch"],"decision":"ask","createdAt":"<REDACTED>"}
```

Deny rules observed:

```json
{"type":"prefix","pattern":["sudo"],"decision":"deny","createdAt":"<REDACTED>"}
{"type":"prefix","pattern":["dd"],"decision":"deny","createdAt":"<REDACTED>"}
{"type":"prefix","pattern":["mkfs"],"decision":"deny","createdAt":"<REDACTED>"}
{"type":"prefix","pattern":["fdisk"],"decision":"deny","createdAt":"<REDACTED>"}
{"type":"prefix","pattern":["chmod"],"decision":"deny","createdAt":"<REDACTED>"}
```

## Agent policy samples

Representative agent overrides:

```json
// Shopify Operator
{"type":"prefix","pattern":["write"],"decision":"allow","justification":"Shopify: file write auto-allowed","createdAt":"<REDACTED>"}
{"type":"prefix","pattern":["apply_patch"],"decision":"ask","justification":"Shopify: patch needs approval","createdAt":"<REDACTED>"}
{"type":"prefix","pattern":["bash"],"decision":"ask","justification":"Shopify: shell exec needs approval","createdAt":"<REDACTED>"}
{"type":"prefix","pattern":["browser"],"decision":"ask","justification":"Shopify: browser needs approval","createdAt":"<REDACTED>"}
```

```json
// 国际站生意助手
{"type":"prefix","pattern":["write"],"decision":"allow","justification":"Merchant assistant: file write auto-allowed","createdAt":"<REDACTED>"}
{"type":"prefix","pattern":["edit"],"decision":"allow","justification":"Merchant assistant: file edit auto-allowed","createdAt":"<REDACTED>"}
{"type":"prefix","pattern":["apply_patch"],"decision":"allow","justification":"Merchant assistant: apply_patch auto-allowed","createdAt":"<REDACTED>"}
{"type":"prefix","pattern":["browser"],"decision":"ask","justification":"Merchant assistant: browser needs approval","createdAt":"<REDACTED>"}
```

## Audit file shape

Sources:

- `/Users/a1-6/.accio/accounts/1758713785/agents/*/permissions/audit.jsonl`

Observed keys:

- `timestamp`
- `agentId`
- `conversationId`
- `toolName`
- `callId`
- `cwd`
- `rwArray`
- `action`
- `reason`
- `automated`
- optional `filePath`
- optional `command`

Observed `action` values in current local logs:

- `auto_allow`

Observed tool names:

- `read`
- `list`
- `cd`
- `web_search`
- `product_supplier_search`
- `sessions_spawn`
- `task_create`
- `task_update`

Representative audit rows:

```json
{
  "timestamp": "<REDACTED>",
  "agentId": "<REDACTED>",
  "conversationId": "<REDACTED>",
  "toolName": "read",
  "callId": "<REDACTED>",
  "cwd": "/Users/a1-6/.accio/accounts/<REDACTED>/agents/<REDACTED>/project/",
  "rwArray": ["/Users/a1-6/.accio/accounts/<REDACTED>/agents/<REDACTED>"],
  "action": "auto_allow",
  "reason": "Execution completed successfully",
  "automated": true,
  "filePath": "/Users/a1-6/.accio/accounts/<REDACTED>/agents/<REDACTED>/agent-core/skills/ecommerce-marketing/SKILL.md"
}
```

```json
{
  "timestamp": "<REDACTED>",
  "agentId": "<REDACTED>",
  "conversationId": "<REDACTED>",
  "toolName": "cd",
  "callId": "<REDACTED>",
  "cwd": "/Users/a1-6/.accio/accounts/<REDACTED>/agents/<REDACTED>/project/",
  "rwArray": ["/Users/a1-6/.accio/accounts/<REDACTED>/agents/<REDACTED>"],
  "action": "auto_allow",
  "reason": "Execution completed successfully",
  "automated": true,
  "command": [
    "bash",
    "-lc",
    "cd \".../agent-core/skills/ecommerce-marketing\";bash scripts/check_environment.sh .../skills .../agent-core/skills"
  ]
}
```

```json
{
  "timestamp": "<REDACTED>",
  "agentId": "<REDACTED>",
  "conversationId": "<REDACTED>",
  "toolName": "sessions_spawn",
  "callId": "<REDACTED>",
  "cwd": "/Users/a1-6/.accio/accounts/<REDACTED>/agents/<REDACTED>/project/",
  "rwArray": ["/Users/a1-6/.accio/accounts/<REDACTED>/agents/<REDACTED>"],
  "action": "auto_allow",
  "reason": "Execution completed successfully",
  "automated": true
}
```

## What the audit log tells us

Useful design signals for an open clone:

1. audit rows record both `decision outcome` and `scope context` (`cwd`, `rwArray`)
2. shell-style tools can log the exact `command` array
3. file tools can log `filePath`
4. agent ID and conversation ID are always attached, so the audit model is already multi-agent aware
