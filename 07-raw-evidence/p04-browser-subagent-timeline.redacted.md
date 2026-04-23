# P0-4 Evidence: Browser Sub-Agent Spawn → Return Timeline

Capture date: `2026-04-23`  
App version: `Accio v0.7.1`

## Main session

- Session:
  `agent:<MAIN_AGENT_ID>:main:cid:<CONVERSATION_ID>`
- User request at `2026-04-23T05:13:01.558Z`:
  search browser for mini car toys

## Spawn event

Main-session `sessions_spawn` call at `2026-04-23T05:13:14.762Z`:

- `agent_id`: `browser`
- `label`: `搜索汽车小玩具产品`
- task:
  search Alibaba for `car toys mini` / `small car toys`, extract `name / price / MOQ / supplier / URL`, return at least `10` products

## Sub-agent session

- Session key:
  `agent:agent:<MAIN_AGENT_ID>:main:cid:<CONVERSATION_ID>:sub:browser:<SUBSESSION_SUFFIX>`
- Agent id: `browser`
- Created: `2026-04-23T05:13:14.767Z`
- Updated: `2026-04-23T05:13:52.370Z`
- Message count: `13`

## Tool chain

Main-session completion record:

```text
tool_chain: browser -> write -> browser -> write -> browser
status: completed
```

Sub-agent milestones from persisted JSONL:

| Timestamp (UTC) | Role | Event |
|---|---|---|
| `2026-04-23T05:13:14.768Z` | `system` | browser-sub-agent prompt loaded |
| `2026-04-23T05:13:14.768Z` | `user` | focused Alibaba extraction task received |
| `2026-04-23T05:13:17.574Z` | `assistant(tool_call)` | `browser(action=tabs)` |
| `2026-04-23T05:13:45.220Z` | `tool` | browser console returned `12` products, saved `products_v2.json` |
| `2026-04-23T05:13:52.363Z` | `assistant` | final success result returned to parent |
| `2026-04-23T05:13:52.370Z` | `tool` | parent `sessions_spawn` receives completion |

## Duration

- Spawn to completion: about `37.6s`
- First browser action after spawn: about `2.8s`
- Data extraction to final return: about `7.1s`

## Output snapshot

Returned status:

- `Success`

Returned data:

- `12` products collected
- `10` summarized in the parent-visible markdown table

Saved files:

- `extract_products_v2.js`
- `products_v2.json`

## Why this matters

This is direct evidence that:

1. sub-agents are persisted as their own sessions
2. browser work is delegated through `sessions_spawn`
3. results are routed back to the main session as a completed task artifact, not as hidden in-process state
