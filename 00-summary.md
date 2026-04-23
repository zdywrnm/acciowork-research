# Accio Work Reverse Analysis Summary

## Current Stage

- Version stamp: `Accio v0.7.1 / 抓取日期 2026-04-22`
- This summary now reflects:
  - `P0-1 应用形态与技术栈指纹`
  - `P0-2 Agent / Skill 数据模型`
  - `P0-3 模型网关` first routing sample
- Current status:
  - `P0-1 complete`
  - `P0-2 main pass complete`
  - `P0-3 partial sample complete`
  - Remaining P0/P1 items are still in progress

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
11. Model traffic is routed through Alibaba’s gateway at `https://phoenix-gw.alibaba.com/api/adk/llm`, with internal model codes such as `1Nexus-*`, `1Orbit-*`, and `1Drift-*` rather than direct vendor model names.
12. Browser control has two paths: preferred `extension relay` and observed `DirectCDP` fallback to Chrome on port `9222`.

## What This Means for Our Clone

- We should treat Accio Work as a serious `desktop runtime`, not just a chat UI.
- Its likely architectural center is: `Electron shell + React UI + internal agent runtime + local persistence + CDP/browser bridge + permission layer`.
- The presence of `node-pty`, `better-sqlite3`, `sqlite-vec`, `sharp`, and `canvas` suggests they leaned into local execution rather than a thin remote client.
- The cleanest clone shape is likely:
  - file-backed agent directories
  - Markdown-native skills
  - account-level shared memory index
  - session-persistent sub-agents
  - separate registration and permission layers

## Evidence to Review First

- Tech stack report: [01-tech-stack.md](/Users/a1-6/research/acciowork/01-tech-stack.md)
- Agent/skill model report: [02-agent-skill-model.md](/Users/a1-6/research/acciowork/02-agent-skill-model.md)
- Bundle structure screenshot: [p0-finder-contents-cropped.png](/Users/a1-6/research/acciowork/06-screenshots/p0-finder-contents-cropped.png)
- Resources screenshot: [p0-finder-resources-cropped.png](/Users/a1-6/research/acciowork/06-screenshots/p0-finder-resources-cropped.png)
- Bundle metadata: [p0-info-plist.txt](/Users/a1-6/research/acciowork/07-raw-evidence/p0-info-plist.txt)
- Extracted package manifest: [p0-package.json](/Users/a1-6/research/acciowork/07-raw-evidence/p0-package.json)
- Runtime snippets: [p02-runtime-snippets.md](/Users/a1-6/research/acciowork/07-raw-evidence/p02-runtime-snippets.md)
- Session topology sample: [p02-session-topology.redacted.md](/Users/a1-6/research/acciowork/07-raw-evidence/p02-session-topology.redacted.md)
- Model routing sample: [p03-routing-sample.redacted.json](/Users/a1-6/research/acciowork/07-raw-evidence/p03-routing-sample.redacted.json)
- Relay handshake sample: [p03-browser-relay-handshake.md](/Users/a1-6/research/acciowork/07-raw-evidence/p03-browser-relay-handshake.md)

## Immediate Next Steps

1. Finish `P0-3`: capture at least one more live model-routing sample from a user-visible selectable model
2. Finish `P0-4`: browser automation path and login/session reuse
3. Finish `P0-5`: permission model from UI + local policy files
4. Start `P1-6 / P1-7 / P1-8`: real walkthroughs, ecosystem, connector matrix
