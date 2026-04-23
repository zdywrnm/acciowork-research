# Accio Work Ecosystem and Integrations

## Stage Status

- App version under analysis: `Accio v0.7.1`
- Capture date: `2026-04-23`
- Scope in this pass: `P1 quick checks on MCP, BYOK clues, and DingTalk public docs`
- Status: `preliminary`

## MCP and Local Extensibility

### Confirmed

1. Accio ships a local CLI:
   [accio-mcp.mjs](/Applications/Accio.app/Contents/Resources/accio-mcp-cli/accio-mcp.mjs:1)
2. That CLI talks to the desktop app’s localhost gateway on port `4097`, not to remote MCP servers directly.
3. The gateway exposes at least:
   - `/mcp/proxy`
   - `/mcp/oauth`
   - `/mcp/custom`
4. The app UI already has a `Custom MCP Servers` entry, so custom MCP registration is a first-class product concept rather than a hidden hook.

Evidence:

- [p03-mcp-cli-entry.md](/Users/a1-6/research/acciowork/07-raw-evidence/p03-mcp-cli-entry.md)

## BYOK / Tenant Override Clues

### What exists in code

The compiled runtime reads these environment variables:

- `ADK_TENANT`
- `ADK_TENANT_OPENAI`
- `ADK_TENANT_GEMINI`
- `ADK_TENANT_CLAUDE`
- `ADK_TENANT_QWEN`
- `ADK_BASE_URL`
- `ADK_IAI_TAG`

### What the quick probe shows

1. These env vars are real runtime inputs, not dead strings.
2. A module-level transport probe with a fake tenant changed the in-memory request object.
3. But the final intercepted HTTP request to `/api/adk/llm/generateContent` did not expose `tenant` in the body, headers, or query when the rest of the request was held constant.

Current interpretation:

- this is not enough to call it a user-facing `BYOK` mode
- it looks more like an internal provider-routing / tenant-override hook than a simple “bring your own API key” backdoor

Evidence:

- [p1-adk-tenant-probe.md](/Users/a1-6/research/acciowork/07-raw-evidence/p1-adk-tenant-probe.md)

## DingTalk Public-Doc Scan

Quick search scope on `2026-04-23`:

- terms around `dingtalk-neulink standard`
- official domains prioritized
- capped as a quick scan, not a deep literature search

Result:

- I did **not** find an authoritative public DingTalk page for the exact term `dingtalk-neulink standard`
- the closest official material found was DingTalk’s general open-platform docs and tutorials, not a specific “NeuLink Standard” connector document

Evidence:

- [p1-dingtalk-neulink-search.md](/Users/a1-6/research/acciowork/07-raw-evidence/p1-dingtalk-neulink-search.md)

## Takeaways

1. Accio’s practical extension surface is already `desktop app + localhost gateway + custom MCP servers`, which is directly relevant to an open clone.
2. The presence of tenant override env vars is interesting, but current evidence does not justify designing our clone around a hidden BYOK backdoor.
3. For DingTalk specifically, we should assume any `neulink standard` wording is internal or undocumented until a public primary source appears.

## Pairings and Channel Approvals

An important adjacent subsystem surfaced in the bundle:

- `pairings-manager`
- `channel.pairings`
- team-member verification errors like `TEAM_CHANNEL_MEMBER_NOT_PAIRED`

Current interpretation:

1. pairings are primarily about `IM/channel identity + permission approval routing`
2. they are used to enrich display names, verify team members, and notify approval requests in IM
3. this is distinct from the MCP/custom-server extension surface
4. I do **not** yet have evidence that pairings are required for the Chrome relay path itself
