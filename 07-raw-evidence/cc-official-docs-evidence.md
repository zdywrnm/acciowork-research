# Claude Code 官方文档证据摘录

抓取日期：2026-04-23（America/Los_Angeles）

## Official URLs

- Claude Code CLI reference: https://code.claude.com/docs/en/cli-usage
- Claude Agent SDK overview: https://code.claude.com/docs/en/agent-sdk/overview
- Claude Agent SDK TypeScript reference: https://platform.claude.com/docs/en/agent-sdk/typescript
- Claude Code MCP: https://code.claude.com/docs/en/mcp
- Claude Code authentication: https://code.claude.com/docs/en/authentication
- Claude Code setup: https://code.claude.com/docs/en/getting-started
- Claude Agent SDK sessions: https://code.claude.com/docs/en/agent-sdk/sessions
- Claude Agent SDK session storage: https://code.claude.com/docs/en/agent-sdk/session-storage
- Claude Code Remote Control: https://code.claude.com/docs/en/remote-control
- Anthropic Claude Code GitHub: https://github.com/anthropics/claude-code
- Zed external agents: https://zed.dev/docs/ai/external-agents

## Facts Used

CLI:

- `claude -p` / `--print` runs non-interactively and exits.
- `--output-format` supports `text`、`json`、`stream-json` in print mode.
- `--json-schema` validates structured JSON output in print mode.
- `--mcp-config` loads MCP server configuration from JSON files or strings.
- `--resume` / `--continue` / `--session-id` support session continuation.
- `--permission-mode` accepts `default`、`acceptEdits`、`plan`、`auto`、`dontAsk`、`bypassPermissions`.

Agent SDK:

- TypeScript SDK installation: `npm install @anthropic-ai/claude-agent-sdk`.
- Main API: `query({ prompt, options })` returns an async generator of SDK messages.
- SDK supports built-in tools, MCP, subagents, permissions, structured output, sessions.
- SDK overview says TypeScript SDK bundles a native Claude Code binary as optional dependency.
- SDK overview says API key setup uses `ANTHROPIC_API_KEY` and that third-party developers should use API key authentication unless approved to offer claude.ai login/rate limits.

MCP:

- MCP scopes: local/user stored in `~/.claude.json`, project stored in `.mcp.json`.
- Project `.mcp.json` is intended for version control and requires approval for security.
- Supported transports include `stdio`、`sse`、`http`.
- `claude mcp add-json` accepts JSON configs for HTTP and stdio servers.
- Claude Code can import MCP servers from Claude Desktop.
- Claude Code can run as an MCP server via `claude mcp serve`, but this exposes Claude Code tools to another MCP client and the client remains responsible for user confirmation.

Auth:

- First `claude` launch opens browser login; user can copy login URL if browser does not open.
- Supported account types include Claude Pro/Max, Team/Enterprise, Console, and cloud providers.
- macOS credentials are stored in encrypted Keychain; Linux/Windows use `~/.claude/.credentials.json` or `$CLAUDE_CONFIG_DIR`.
- Auth precedence includes cloud provider env, `ANTHROPIC_AUTH_TOKEN`, `ANTHROPIC_API_KEY`, `apiKeyHelper`, long-lived OAuth token, then subscription OAuth credentials.

Sessions:

- SDK session history persists automatically and contains prompt, tool calls/results, and responses.
- Resume by session id is supported; session files can be moved across hosts if restored to `~/.claude/projects/<encoded-cwd>/<session-id>.jsonl` with matching cwd.
- SessionStore can mirror JSONL transcript entries to external storage, but the Claude Code subprocess writes local disk first.

Install:

- Recommended install uses native installer `curl -fsSL https://claude.ai/install.sh | bash`.
- Homebrew, WinGet, apt/dnf/apk are supported.
- GitHub README marks npm installation deprecated, but setup docs still document `npm install -g @anthropic-ai/claude-code`.
- Setup docs say the npm package installs the same native binary via per-platform optional dependency, and the installed `claude` binary does not itself invoke Node.

Remote Control:

- Remote Control connects local Claude Code sessions to claude.ai/code/mobile.
- It runs as a local process and uses outbound HTTPS; it does not open inbound ports.
- It requires claude.ai subscription OAuth and does not support API key auth.
- It is not a local HTTP/IPC API for third-party desktop apps.

Zed:

- Zed supports external agents via ACP, including Claude Agent.
- Zed Claude Agent path runs Claude Agent SDK / Claude Code through a dedicated adapter.
- Zed docs say Claude Agent authentication is decoupled from Zed model provider settings and can use API key or “Log in with Claude Code”.
- Zed docs also note some Agent Panel features are not available with external agents.
