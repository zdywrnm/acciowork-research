# Accio Work Reverse Analysis Summary

## Current Stage

- Version stamp: `Accio v0.7.1 / 抓取日期 2026-04-23`
- This summary now reflects:
  - `P0-1 应用形态与技术栈指纹`
  - `P0-2 Agent / Skill 数据模型`
  - `P0-3 模型网关`
  - `P0-4 浏览器自动化` initial pass + limited retry
  - `P0-5 权限模型`
  - `P1` quick checks on MCP/BYOK and DingTalk public docs
- Current status:
  - `P0-1 complete`
  - `P0-2 main pass complete`
  - `P0-3 complete`
  - `P0-4 fallback path complete; positive relay handshake still incomplete`
  - `P0-5 complete`
  - `P1-7 / P1-8 partial`
  - `P1 live tools schema sample complete`

## Key Judgments

1. `Accio Work` on this Mac is installed as `/Applications/Accio.app`, bundle id `com.accio.desktop`, version `0.7.1`.
2. It is definitively an `Electron` desktop app, not Tauri or native Swift.
3. Electron version is `35.7.5`.
4. The renderer is `React`; the product shell is built around internal `@phoenix/*` packages.
5. The bundle carries native modules for `terminal exec`, `SQLite/vector retrieval`, `image processing`, and `security guard` capabilities.
6. The app ships with a bundled `Chrome extension` and `CDP`-related main-process code, which strongly suggests browser attachment rather than only an embedded browser.
7. The update path is Electron-native, using `electron-updater` plus `Squirrel.framework`, with a `beta` channel hosted at `work-download.accio-ai.com`.
8. A local agent is a file-backed runtime unit with `profile + prompt scaffold + tool registry + permission policy + sessions + project dir + optional local memory index`.
9. Skills are primarily packaged as `Markdown skill bundles`, while plugin-backed agents add `agent.json + agents.md + connectors.json + policy rules`.
10. Team/sub-agent execution is real `multi-session persistence`, but current evidence points to `same-process orchestration`, not separate OS worker processes.
11. `accio-mcp.mjs` is not a standalone server. It is a local CLI client to the desktop app’s localhost gateway at `http://127.0.0.1:4097/mcp/*`.
12. The local gateway really listens on port `4097`, but enforces a loopback-only policy in application logic: `127.0.0.1` returns health `200`, LAN IP requests return `403 Forbidden`.
13. Model traffic is routed through Alibaba’s gateway at `https://phoenix-gw.alibaba.com/api/adk/llm`, with internal model codes such as `1Nexus-*`, `1Orbit-*`, and `1Drift-*` rather than direct vendor model names.
14. A live `Nexus / Orbit / Drift` comparison shows the client emits the same main `systemInstruction` hash and the same `tools` hash for all three families. Model switching changed the model code, but not the client-side system prompt or tool list.
15. The desktop bundle contains a native `security_guard.node` addon. `sg_k` is attached client-side before the network request leaves the machine, via a security-guard transport interceptor.
16. In module-level transport interception, changing `tenant` changes the in-memory request object, but does not change the final `/api/adk/llm/generateContent` HTTP body, headers, or `sg_k` when the rest of the request is held constant.
17. Browser control has two paths: preferred `extension relay` and observed `DirectCDP` fallback to Chrome on port `9222`.
18. The fallback path is positively observed in runtime logs. A temporary Chrome profile loaded with the bundled relay extension did not complete the relay handshake during this round, so relay runtime validation remains partial.
19. Browser sub-agents are true persisted sessions with their own prompt, tool chain, and timestamps. One captured Alibaba task ran from `2026-04-23T05:13:14.767Z` to `2026-04-23T05:13:52.370Z`, using `browser -> write -> browser -> write -> browser`.
20. No explicit rate-limit or remaining-quota fields were observed in the captured `adk.llm.response` / `llm.call` traces. Credit/quota UX appears to come from separate entitlement endpoints such as `/api/entitlement/quota`, not from the LLM response body itself.
21. The permission model is a three-layer stack, not a single four-level switch: `policy decision (allow/ask/deny)` + `user approval result (approved / approved_always / denied / denied_always / abort)` + `sandbox scope (read_only / read_write_cwd / danger_full_access)`.
22. The current desktop UI exposes only two top-level execution modes: `默认权限` and `完全访问权限`; I did not find a separate user-facing `restricted` mode in this build.
23. Account-level `policy.jsonl` currently has `120` prefix rules: `104 allow`, `6 ask`, `10 deny`, with `43` rules carrying `bypassSandbox=true`.
24. Agent-level policy overrides are real and materially different by agent. `Shopify Operator` asks for `bash/browser/apply_patch`, while `Accio` and `Coder` auto-allow `write/edit/apply_patch`.
25. On macOS, terminal/file execution is enforced through real seatbelt sandboxing via `/usr/bin/sandbox-exec`, not only by soft policy checks.
26. `danger_full_access` expands the seatbelt file-write policy to `^/`, while `read_write_cwd` constrains writes to derived writable roots.
27. The unmatched-command fallback in the permission engine is currently `allow`; safety then depends on sandbox scope unless the matched rule also carries `bypassSandbox=true`.
28. `pairings-manager` is an IM/team pairing and approval-routing subsystem. It gates team conversations, enriches IM identities, and routes permission approvals over channels; no direct browser-relay coupling is proven yet.
29. A live `32-tool` LLM request body was captured and redacted. The tool array confirms that Accio sends a full runtime tool schema to the gateway, including file, shell, browser-orchestration, task, memory, image, Gmail-listener, and product-supplier tools.

## What This Means for Our Clone

- We should treat Accio Work as a serious `desktop runtime`, not just a chat UI.
- Its likely architectural center is: `Electron shell + React UI + internal agent runtime + local persistence + CDP/browser bridge + permission layer`.
- The closest open-source analogue is not “one assistant with tools”, but `desktop orchestrator + local gateway + persisted multi-session agent runtime`.
- The presence of `node-pty`, `better-sqlite3`, `sqlite-vec`, `sharp`, and `canvas` suggests they leaned into local execution rather than a thin remote client.
- The cleanest clone shape is likely:
  - file-backed agent directories
  - Markdown-native skills
  - account-level shared memory index
  - session-persistent sub-agents
  - separate registration and permission layers
  - localhost gateway for MCP/custom tools
  - browser bridge abstraction that can support both `relay` and `direct CDP`

## Evidence to Review First

- Tech stack report: [01-tech-stack.md](/Users/a1-6/research/acciowork/01-tech-stack.md)
- Agent/skill model report: [02-agent-skill-model.md](/Users/a1-6/research/acciowork/02-agent-skill-model.md)
- Task walkthrough seed log: [03-task-walkthrough.md](/Users/a1-6/research/acciowork/03-task-walkthrough.md)
- Ecosystem / MCP note: [05-ecosystem.md](/Users/a1-6/research/acciowork/05-ecosystem.md)
- Bundle structure screenshot: [p0-finder-contents-cropped.png](/Users/a1-6/research/acciowork/06-screenshots/p0-finder-contents-cropped.png)
- Resources screenshot: [p0-finder-resources-cropped.png](/Users/a1-6/research/acciowork/06-screenshots/p0-finder-resources-cropped.png)
- Bundle metadata: [p0-info-plist.txt](/Users/a1-6/research/acciowork/07-raw-evidence/p0-info-plist.txt)
- Extracted package manifest: [p0-package.json](/Users/a1-6/research/acciowork/07-raw-evidence/p0-package.json)
- Runtime snippets: [p02-runtime-snippets.md](/Users/a1-6/research/acciowork/07-raw-evidence/p02-runtime-snippets.md)
- Session topology sample: [p02-session-topology.redacted.md](/Users/a1-6/research/acciowork/07-raw-evidence/p02-session-topology.redacted.md)
- Model routing sample: [p03-routing-sample.redacted.json](/Users/a1-6/research/acciowork/07-raw-evidence/p03-routing-sample.redacted.json)
- Relay handshake sample: [p03-browser-relay-handshake.md](/Users/a1-6/research/acciowork/07-raw-evidence/p03-browser-relay-handshake.md)
- MCP CLI evidence: [p03-mcp-cli-entry.md](/Users/a1-6/research/acciowork/07-raw-evidence/p03-mcp-cli-entry.md)
- Live family compare: [p03-live-model-families.redacted.md](/Users/a1-6/research/acciowork/07-raw-evidence/p03-live-model-families.redacted.md)
- `sg_k` evidence: [p03-sgk-security-guard.md](/Users/a1-6/research/acciowork/07-raw-evidence/p03-sgk-security-guard.md)
- Tenant probe: [p1-adk-tenant-probe.md](/Users/a1-6/research/acciowork/07-raw-evidence/p1-adk-tenant-probe.md)
- Browser dual-path evidence: [p04-browser-dual-path.md](/Users/a1-6/research/acciowork/07-raw-evidence/p04-browser-dual-path.md)
- Browser sub-agent timing: [p04-browser-subagent-timeline.redacted.md](/Users/a1-6/research/acciowork/07-raw-evidence/p04-browser-subagent-timeline.redacted.md)
- Permission picker screenshot: [p05-ui-permission-picker.png](/Users/a1-6/research/acciowork/06-screenshots/p05-ui-permission-picker.png)
- Permission model: [p05-permission-model.md](/Users/a1-6/research/acciowork/07-raw-evidence/p05-permission-model.md)
- Policy/audit samples: [p05-policy-audit-samples.redacted.md](/Users/a1-6/research/acciowork/07-raw-evidence/p05-policy-audit-samples.redacted.md)
- Sandbox engine: [p05-sandbox-engine.md](/Users/a1-6/research/acciowork/07-raw-evidence/p05-sandbox-engine.md)
- Pairings manager: [p05-pairings-manager.md](/Users/a1-6/research/acciowork/07-raw-evidence/p05-pairings-manager.md)
- Limited relay retry: [p04-relay-retry-limited.md](/Users/a1-6/research/acciowork/07-raw-evidence/p04-relay-retry-limited.md)
- Live 32-tool request body: [p1-live-tools-schema.redacted.json](/Users/a1-6/research/acciowork/07-raw-evidence/p1-live-tools-schema.redacted.json)

## Immediate Next Steps

1. Expand `P1-6`: two more real scenario walkthroughs besides the Alibaba browser run
2. Expand `P1-7 / P1-8`: connector matrix and marketplace/developer workflow
3. Treat relay positive-handshake proof as optional follow-up unless new product-side pairing state becomes available
