# Accio Weaknesses = Our Opportunities

Version stamp: `Accio v0.7.1 / capture date 2026-04-23`

## 1. Permission Semantics Are Blurred

Accio weakness:

The UI exposes `默认权限` and `完全访问权限`, but the runtime actually has separate layers: policy decision, user approval outcome, and sandbox scope. A user cannot easily tell whether a tool is allowed because of policy, approval cache, sandbox breadth, or `bypassSandbox`.

Our opportunity:

Expose these as separate concepts:

- `Policy`: allow, ask, deny
- `Approval`: once, always, denied, expired
- `Sandbox`: read-only, project-write, full-disk
- `Audit`: visible record of why a tool ran

MVP move:

Build a permission inspector that explains every tool call in plain language: `who asked`, `what it touches`, `why it is allowed`, and `what sandbox applies`.

## 2. Default Command Policy Is Too Broad

Accio weakness:

The unmatched-command path falls through to `allow`; safety then relies on sandbox scope unless a prefix rule catches the command. This is pragmatic, but it is hard to reason about and risky when rules carry `bypassSandbox=true`.

Our opportunity:

Start stricter:

- unknown shell commands default to `ask`
- file writes outside the project default to `ask`
- destructive command families default to `deny`
- per-agent overrides are visible and reviewable

MVP move:

Ship a policy engine with declarative rules, a simulator, and a diff view before saving permission changes.

## 3. Browser Connection Health Is Opaque

Accio weakness:

The bundle clearly implements a Chrome relay extension, but the tested environment still fell back to direct CDP on `9222`. The UI does not make the transport state obvious enough for debugging.

Our opportunity:

Make browser automation observable:

- show current transport: extension relay, direct CDP, embedded browser, or unavailable
- show why fallback happened
- show controlled tabs and retained tabs
- provide a one-click diagnostic report

MVP move:

Treat browser control as a pluggable adapter with explicit status, logs, and user-facing repair actions.

## 4. Pairing And Remote Approval Are Hard To Understand

Accio weakness:

`pairings-manager` gates IM/team flows and permission approvals, but the local state is sparse and the relationship between channel pairing, team membership, and approval routing is not obvious.

Our opportunity:

Make pairing a separate product surface:

- identities
- channels
- groups
- approval authority
- expiry and revoke controls

MVP move:

Model pairings as auditable grants, not hidden connector side effects.

## 5. Model Routing Is Centralized And Opaque

Accio weakness:

UI model choices map to internal `1Nexus-*`, `1Orbit-*`, and `1Drift-*` codes behind Alibaba's gateway. Live traces show the same client prompt/tool schema across families, but provider identity, routing logic, and quota treatment are not transparent.

Our opportunity:

Make routing boring and inspectable:

- provider
- model
- tenant/key source
- fallback chain
- prompt hash
- tool schema hash
- token and cost estimate

MVP move:

Use an explicit model router config and show the resolved provider before each run.

## 6. BYOK Is Not A Product-Grade Path

Accio weakness:

The bundle contains tenant override environment variables, but the probe did not prove a clean user-facing BYOK mode. It looks more like internal tenant routing than a reliable extension point.

Our opportunity:

Make BYOK a first-class path:

- OpenAI-compatible endpoint
- Anthropic-compatible endpoint
- Gemini-compatible endpoint
- local model endpoint
- per-agent model policy

MVP move:

Design provider credentials as local encrypted config with per-agent routing rules and no hidden tenant magic.

## 7. Tool Schemas Are Powerful But Heavy

Accio weakness:

The live request sends a full `32`-tool schema to the gateway. This is flexible, but it increases prompt surface area and makes tool availability harder to audit.

Our opportunity:

Use scoped tool loading:

- base tools
- agent tools
- skill tools
- connector tools
- task-specific tools

MVP move:

Log tool schema hashes per turn and show which skill or connector injected each tool.

## 8. Local Data Is Rich But Not User-Legible

Accio weakness:

Agent directories, sessions, vector memory, policies, plugin state, and connector state are all present locally, but the user-facing product does not make the data model easy to inspect or export.

Our opportunity:

Make local ownership a headline feature:

- open agent folder
- export session
- inspect memory
- prune vectors
- view plugin files
- back up account state

MVP move:

Use file-backed state intentionally and provide a built-in data browser for agents, skills, memories, sessions, and audits.

## 9. Skill Ecosystem Is Strong But Closed-Feeling

Accio weakness:

Skills are Markdown-friendly under the hood, but marketplace submission, review, versioning, and developer workflow are not yet clear from the local product.

Our opportunity:

Make skills Git-native:

- `SKILL.md`
- `skill.json`
- examples
- tests
- permissions
- required connectors
- changelog

MVP move:

Ship a local skill SDK and validation CLI before building a marketplace.

## 10. Quota And Pricing Signals Are Fragmented

Accio weakness:

LLM traces expose token usage, while visible credit and exhaustion messages come from separate entitlement/config surfaces. This makes cost and failure diagnosis harder.

Our opportunity:

Unify usage reporting:

- tokens
- tool calls
- browser minutes
- connector calls
- image generation
- quota source
- retry/fallback cost

MVP move:

Put a per-run cost ledger beside every agent run, even if the first version only estimates.

## Product Positioning Takeaway

Do not compete with Accio by copying every connector first. Compete by making the runtime understandable.

The open clone's wedge should be:

- transparent local data
- explainable permissions
- inspectable tool schemas
- pluggable model routing
- debuggable browser automation
- Git-native skills
- audit logs designed for operators
