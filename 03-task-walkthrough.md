# Accio Work Task Walkthrough

## Stage Status

- App version under analysis: `Accio v0.7.1`
- Capture date: `2026-04-23`
- Scope in this document: `P0-4 initial seed walkthrough`
- Status: `initial pass complete`

## Walkthrough A: Browser Sub-Agent Alibaba Search

### Task Type

- Category: `read-only browser automation`
- External side effects: `none`
- Completion boundary: full run allowed under the user’s safety rules

### User Prompt

Main-session user prompt:

- `打开浏览器搜搜汽车小玩具的品`

Primary agent then normalized it into a browser sub-agent task:

- Search Alibaba for `car toys mini` / `small car toys`
- Extract `product name`, `price`, `MOQ`, `supplier`, `product URL`
- Return at least `10` results

### Session Topology

- Main session:
  `agent:<MAIN_AGENT_ID>:main:cid:<CONVERSATION_ID>`
- Spawned sub-agent:
  `agent:agent:<MAIN_AGENT_ID>:main:cid:<CONVERSATION_ID>:sub:browser:<SUBSESSION_SUFFIX>`
- Sub-agent label: `搜索汽车小玩具产品`
- Evidence:
  [p04-browser-subagent-timeline.redacted.md](/Users/a1-6/research/acciowork/07-raw-evidence/p04-browser-subagent-timeline.redacted.md)

### Timeline

| Time (UTC) | Event |
|---|---|
| `2026-04-23T05:13:01.558Z` | Main session receives the user request |
| `2026-04-23T05:13:14.762Z` | Main session emits `sessions_spawn` for `agent_id="browser"` |
| `2026-04-23T05:13:14.767Z` | Browser sub-agent session is created |
| `2026-04-23T05:13:17.574Z` | First browser tool call: `action=tabs` |
| `2026-04-23T05:13:45.220Z` | Browser console extractor returns `12` Alibaba products and saves `products_v2.json` |
| `2026-04-23T05:13:52.363Z` | Sub-agent returns final structured result |
| `2026-04-23T05:13:52.370Z` | Main session records `sessions_spawn` completion |

Observed sub-agent runtime:

- Start: `2026-04-23T05:13:14.767Z`
- End: `2026-04-23T05:13:52.370Z`
- Elapsed: about `37.6s`
- Tool chain: `browser -> write -> browser -> write -> browser`

### What The Browser Sub-Agent Actually Did

From the persisted session JSONL:

1. read the browser-sub-agent system prompt
2. received one focused Alibaba extraction task
3. called `browser(action=tabs)` first, which matches the prompt’s tab-reuse rule
4. wrote an extraction helper script into the agent project directory
5. executed the script in the browser console against the target tab
6. wrote raw results to `products_v2.json`
7. returned a structured markdown table to the main session

### Output Quality

Observed result:

- `12` raw products extracted
- `10` summarized back to the user
- fields included `name`, `price`, `MOQ`, `supplier`, `URL`
- raw data saved locally in the agent project directory

This is a legitimate end-to-end browser-agent workflow, not a mocked “tool description” answer.

### Design Signals For Our Clone

1. The browser worker has its own focused system prompt, distinct from the main agent.
2. `sessions_spawn` is the orchestration primitive; the main agent does not directly inline browser logic.
3. The worker is optimized for short, bounded jobs and explicitly told to stop early on repeated failures.
4. Agent-opened tabs persist beyond a single sub-agent turn, which implies session-level browser state reuse.

### Risks / Gaps

1. This run validated the `DirectCDP`-capable browser worker path, but not a positive relay-extension handshake.
2. The extracted result quality was good enough for commodity product search, but I have not yet benchmarked more brittle flows such as login-gated dashboards or pagination-heavy tasks.

### Unfinished Action List

- none for this specific walkthrough
