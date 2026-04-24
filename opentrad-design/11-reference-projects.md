# OpenTrad 参考项目速扫

抓取日期：2026-04-23（America/Los_Angeles）  
目标：只看架构和可偷设计，不做源码深挖。证据见 `../07-raw-evidence/cc-reference-projects-evidence.md`。

## Cline

Cline 是 VS Code extension，主仓库 package 显示 TypeScript + VS Code extension 形态，入口 `dist/extension.js`，核心依赖包括 Anthropic/OpenAI SDK、Playwright、VS Code API。它的 provider 抽象是 BYOK 典型：Anthropic、OpenAI、Gemini、Bedrock、Vertex、OpenRouter、Ollama/LM Studio/OpenAI-compatible 都可接入，并在任务循环里显示 token/cost。可偷的是三件事：人类逐步批准文件修改/终端命令的安全 UX，workspace checkpoint/restore 机制，以及 MCP 作为“让 agent 自己扩工具”的生态入口。OpenTrad 不需要复制 VS Code 绑定，但应复制它的权限确认、diff 审阅和成本透明度。

Sources: [GitHub](https://github.com/cline/cline)

## Open WebUI

Open WebUI 是自托管 LLM Web 平台，前端是 SvelteKit/Vite/Tailwind，后端 requirements 明确是 FastAPI/Uvicorn/SQLAlchemy，并集成 OpenAI、Anthropic、Google、LangChain、MCP、Playwright、多个向量库和 RAG 工具。它的 provider 抽象偏“LLM 控制台”：Ollama 和 OpenAI-compatible 是一等公民，支持多 provider、RAG、Python function tool、RBAC、OAuth/LDAP/SSO。可偷的是 provider/admin 配置模型、RAG/知识库接入、工具 marketplace 的后台治理；但它更像服务器产品，桌面本地工作台不应照搬复杂权限和多租户。

Sources: [GitHub](https://github.com/open-webui/open-webui)

## Cherry Studio

Cherry Studio 是 Electron + React/TypeScript 桌面客户端，package 显示 `electron-vite`、`electron-builder`、React 19、TanStack Query、Drizzle、Dexie、i18n，并已依赖 `@anthropic-ai/claude-agent-sdk`。产品定位是中文 AI productivity studio，强调 300+ assistants、多模型统一入口、知识库、MCP、企业版权限和私有部署。Provider 抽象偏“国产/国际模型大全 + 本地客户端配置”，对 OpenTrad 的价值是中文场景、模型配置 UI、桌面更新/本地数据库、MCP 管理页。注意其 AGPL-3.0，代码复用要严格处理许可证。

Sources: [GitHub](https://github.com/CherryHQ/cherry-studio)

## LibreChat

LibreChat 是 ChatGPT-like 自托管 Web app，monorepo 分 `api`、`client`、`packages/*`；前端 React/Vite/Radix/TanStack，后端 Express/Mongoose/Redis/Passport，数据库主要围绕 MongoDB。Provider 抽象覆盖 Anthropic、OpenAI、Azure、Bedrock、Google/Vertex、OpenRouter、DeepSeek、Qwen、Ollama 等，并支持 custom endpoints、agents、MCP tools、code interpreter、artifacts、Web search。可偷的是 YAML 配置化 endpoint、agent marketplace/share 模型、多用户 auth 和 conversation schema；OpenTrad v1 不需要多用户，但可借鉴其“endpoint/provider 与 agent 配置分离”的数据模型。

Sources: [GitHub](https://github.com/danny-avila/LibreChat)

## Zed Agent Panel

Zed 是 Rust/GPUI 编辑器，Agent Panel 是原生 AI 工作区：thread sidebar、并行 agents、worktree isolation、checkpoint、tool permission profiles（Write/Ask/Minimal）、模型切换和 token usage。Provider 抽象同时支持 Zed hosted models、BYOK providers、OpenAI-compatible、Ollama，以及 external agents。关键证据是 Zed external agents：它通过 ACP 跑 Gemini CLI、Claude Agent、Codex；Claude Agent 路径会安装 managed `@zed-industries/claude-agent-acp`，并可通过 `/login` 选择 API key 或 “Log in with Claude Code”。OpenTrad 可偷的是 ACP 方向、thread/worktree UX、profile 化工具权限；但 v1 不应依赖 Zed 的托管 adapter 能力。

Sources: [Agent Panel](https://zed.dev/docs/ai/agent-panel), [External Agents](https://zed.dev/docs/ai/external-agents), [LLM Providers](https://zed.dev/docs/ai/llm-providers)

## 横向启发

| 维度 | 最值得参考 | 对 OpenTrad 的取舍 |
| --- | --- | --- |
| CLI/agent 嵌入 | Zed external agents、opcode/community wrappers | v1 直接包装本机 Claude CLI，未来再评估 ACP |
| 权限 UX | Cline、Zed profiles | P0 必须有“只读/草稿/执行前确认”三档 |
| Provider 抽象 | LibreChat、Open WebUI、Cherry Studio | v1 暂不做多模型，先保留接口 |
| 中文/国内模型体验 | Cherry Studio | v2 再做 Qwen/DeepSeek/硅基流动等 |
| RAG/知识库 | Open WebUI | v1 只做本地文件/商品资料，避免建复杂知识库 |
| 多用户/企业 | LibreChat、Open WebUI | v1 桌面单用户，不做 |
| 会话/并行任务 | Zed thread sidebar | P0 建 session list；worktree/并行可 P1 |
