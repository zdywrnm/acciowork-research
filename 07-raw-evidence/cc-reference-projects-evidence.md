# OpenTrad 参考项目证据摘录

抓取日期：2026-04-23（America/Los_Angeles）

## Source URLs

- Cline: https://github.com/cline/cline
- Open WebUI: https://github.com/open-webui/open-webui
- Cherry Studio: https://github.com/CherryHQ/cherry-studio
- LibreChat: https://github.com/danny-avila/LibreChat
- Zed Agent Panel: https://zed.dev/docs/ai/agent-panel
- Zed LLM Providers: https://zed.dev/docs/ai/llm-providers
- Zed External Agents: https://zed.dev/docs/ai/external-agents
- Zed Parallel Agents: https://zed.dev/docs/ai/parallel-agents

## Package / Stack Probes

### Cline

```json
{
  "name": "claude-dev",
  "version": "3.80.0",
  "main": "./dist/extension.js",
  "engines": { "vscode": "^1.84.0" },
  "dependencies": [
    "@anthropic-ai/sdk",
    "@anthropic-ai/vertex-sdk",
    "@playwright/test",
    "@vscode/codicons",
    "axios",
    "openai",
    "vscode-uri"
  ],
  "devDependencies": ["@types/vscode", "@vscode/vsce", "esbuild", "typescript"]
}
```

GitHub README states Cline can create/edit files, execute terminal commands, use browser, and use MCP, with human approval for file changes and commands.

### Open WebUI

```json
{
  "name": "open-webui",
  "version": "0.9.1",
  "type": "module",
  "engines": { "node": ">=18.13.0 <=22.x.x", "npm": ">=6.0.0" },
  "frontend": ["SvelteKit", "Vite", "Tailwind", "xterm", "CodeMirror"],
  "backend": ["FastAPI", "Uvicorn", "SQLAlchemy", "aiosqlite", "asyncpg"],
  "ai_libs": ["mcp", "openai", "anthropic", "google-genai", "langchain", "chromadb", "qdrant-client", "playwright"]
}
```

README states Open WebUI supports Ollama/OpenAI-compatible APIs, RAG, RBAC, native Python function calling, web search, vector databases, and multiple storage options.

### Cherry Studio

```json
{
  "name": "CherryStudio",
  "version": "1.9.3",
  "main": "./out/main/index.js",
  "engines": { "node": ">=24.11.1" },
  "stack": ["Electron", "electron-vite", "electron-builder", "React", "TypeScript", "Drizzle", "Dexie", "TanStack Query"],
  "notable_dependencies": ["@anthropic-ai/claude-agent-sdk"]
}
```

GitHub README describes Cherry Studio as AI productivity studio with smart chat, autonomous agents, 300+ assistants, unified access to frontier LLMs, and enterprise features.

### LibreChat

```json
{
  "name": "LibreChat",
  "version": "v0.8.5",
  "workspaces": ["api", "client", "packages/*"],
  "client": ["React", "Vite", "Radix", "TanStack Query", "@mcp-ui/client"],
  "api": ["Express", "Mongoose", "Redis", "Passport", "OpenAI", "@langchain/core"]
}
```

GitHub README lists model providers including Anthropic, AWS Bedrock, OpenAI, Azure, Google, Vertex, custom OpenAI-compatible endpoints, Ollama, Groq, Cohere, Mistral, OpenRouter, DeepSeek, Qwen, plus Agents, MCP tools, code interpreter, artifacts, and web search.

### Zed

Docs facts:

- Agent Panel lets users interact with agents that read/write/run code.
- First use needs at least one LLM provider or external agent.
- Agent Panel supports multiple independent threads, thread history, worktree isolation, checkpoints, model switching, token usage, and tool profiles.
- Built-in profiles include `Write`, `Ask`, `Minimal`.
- LLM providers support hosted Zed models, Anthropic/OpenAI/Google/Mistral/DeepSeek/Ollama, OpenAI-compatible APIs, and BYOK stored in OS credential storage.
- External agents use ACP; Claude Agent is run through managed `@zed-industries/claude-agent-acp`, can use Claude Code login, and has some unsupported features compared with Zed native agent.
