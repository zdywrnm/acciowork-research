# Claude Code 本地 CLI 探测证据（脱敏）

日期：2026-04-23（America/Los_Angeles）  
机器：macOS，本机用户路径已保留到项目级；账号、邮箱、组织、session id、uuid 均已脱敏。

## Commands

```bash
command -v claude
claude --version
claude --help
claude auth status --text
claude auth --help
claude auth login --help
claude mcp --help
claude mcp add --help
claude mcp serve --help
claude remote-control --help
claude -p --output-format json --tools "" --no-chrome --max-turns 1 "Return exactly OK."
claude -p --output-format stream-json --verbose --tools "" --no-chrome --max-turns 1 --model haiku "Return exactly OK."
find ~/.claude/projects/-Users-a1-6-research-acciowork -maxdepth 2 -type f
```

## Version / Auth

```text
command -v claude
/Users/a1-6/.claude/bin/claude

claude --version
2.1.119 (Claude Code)

claude auth status --text
Login method: Claude Pro account
Organization: <REDACTED>
Email: <REDACTED>
```

## Selected CLI Flags

`claude --help` 本地输出确认以下关键 flags：

```text
-p, --print
--output-format <format>        text | json | stream-json
--input-format <format>         text | stream-json
--json-schema <schema>
--mcp-config <configs...>
--strict-mcp-config
-c, --continue
-r, --resume [value]
--session-id <uuid>
--no-session-persistence
--permission-mode <mode>        acceptEdits | auto | bypassPermissions | default | dontAsk | plan
--allowedTools / --allowed-tools
--disallowedTools / --disallowed-tools
--tools <tools...>
--model <model>
--fallback-model <model>
--max-budget-usd <amount>
--max-turns <number>
--debug-file <path>
--bare
--chrome / --no-chrome
```

## MCP Help

```text
claude mcp
Commands:
  add [options] <name> <commandOrUrl> [args...]
  add-from-claude-desktop [options]
  add-json [options] <name> <json>
  get <name>
  list
  remove [options] <name>
  reset-project-choices
  serve [options]

claude mcp add
Options:
  --transport <transport>  stdio, sse, http
  --scope <scope>          local, user, project
  --env <env...>
  --header <header...>
  --client-id <clientId>
  --client-secret
  --callback-port <port>

claude mcp serve
Start the Claude Code MCP server
```

## Auth Login Help

```text
claude auth login
Options:
  --claudeai       Use Claude subscription (default)
  --console        Use Anthropic Console (API usage billing)
  --email <email>  Pre-populate email address on the login page
  --sso            Force SSO login flow
```

## Remote Control Help

```text
claude remote-control
Remote Control - Connect your local environment to claude.ai/code

Options:
  --name <name>
  --remote-control-session-name-prefix <prefix>
  --permission-mode <mode>
  --spawn <mode>     same-dir | worktree | session
  --capacity <N>

Notes:
  - You must be logged in with a Claude account that has a subscription
  - Run `claude` first in the directory to accept the workspace trust dialog
```

## Non-interactive JSON Sample

Command:

```bash
claude -p --output-format json --tools "" --no-chrome --max-turns 1 "Return exactly OK."
```

Redacted shape:

```json
{
  "type": "result",
  "subtype": "success",
  "is_error": false,
  "api_error_status": null,
  "duration_ms": 1887,
  "duration_api_ms": 2707,
  "num_turns": 1,
  "result": "OK",
  "stop_reason": "end_turn",
  "session_id": "<REDACTED_UUID>",
  "total_cost_usd": 0.03571175,
  "usage": {
    "input_tokens": 6,
    "cache_creation_input_tokens": 5377,
    "cache_read_input_tokens": 3057,
    "output_tokens": 6,
    "server_tool_use": {
      "web_search_requests": 0,
      "web_fetch_requests": 0
    },
    "service_tier": "standard",
    "iterations": [
      {
        "input_tokens": 6,
        "output_tokens": 6,
        "type": "message"
      }
    ],
    "speed": "standard"
  },
  "modelUsage": {
    "<MODEL_NAME>": {
      "inputTokens": 6,
      "outputTokens": 6,
      "costUSD": 0.03531475,
      "contextWindow": 1000000,
      "maxOutputTokens": 64000
    }
  },
  "permission_denials": [],
  "terminal_reason": "completed",
  "fast_mode_state": "off",
  "uuid": "<REDACTED_UUID>"
}
```

## Stream JSON Sample

Command:

```bash
claude -p --output-format stream-json --verbose --tools "" --no-chrome --max-turns 1 --model haiku "Return exactly OK."
```

Observed NDJSON event types:

```text
rate_limit_event
system/init
assistant
assistant
result
```

Redacted `rate_limit_event` shape:

```json
{
  "type": "rate_limit_event",
  "rate_limit_info": {
    "status": "allowed",
    "resetsAt": 1777028400,
    "rateLimitType": "five_hour",
    "overageStatus": "rejected",
    "overageDisabledReason": "out_of_credits",
    "isUsingOverage": false
  },
  "uuid": "<REDACTED_UUID>",
  "session_id": "<REDACTED_UUID>"
}
```

Redacted `system/init` shape:

```json
{
  "type": "system",
  "subtype": "init",
  "cwd": "/Users/a1-6/research/acciowork",
  "session_id": "<REDACTED_UUID>",
  "tools": ["<MCP_OR_BUILTIN_TOOL_NAMES_REDACTED>"],
  "mcp_servers": [{"name": "<REDACTED>", "status": "needs-auth"}],
  "model": "claude-haiku-4-5-20251001",
  "permissionMode": "default",
  "slash_commands": ["<REDACTED_LIST>"],
  "apiKeySource": "none",
  "claude_code_version": "2.1.119",
  "output_style": "default",
  "agents": ["<REDACTED_LIST>"],
  "skills": ["<REDACTED_LIST>"],
  "plugins": [],
  "analytics_disabled": false,
  "uuid": "<REDACTED_UUID>",
  "memory_paths": {
    "auto": "/Users/a1-6/.claude/projects/-Users-a1-6-research-acciowork/memory/"
  },
  "fast_mode_state": "off"
}
```

Assistant event can contain separate content blocks, including text and, in this run, a thinking block. Thinking text/signature were not retained in this evidence file.

## Session Files

After the two test calls, Claude Code created local transcripts:

```text
~/.claude/projects/-Users-a1-6-research-acciowork/<REDACTED_UUID>.jsonl
~/.claude/projects/-Users-a1-6-research-acciowork/<REDACTED_UUID>.jsonl
```

No transcript content or credential file content was read.

## Local Installation Footprint

```text
~/.claude/bin/claude: 4.3 KB shell script shim
~/.claude: 39 MB
~/.local/share/claude: 792 MB on this machine
```

The 792 MB figure is local cache/history/runtime footprint on this machine and should not be used as the clean installer size.
