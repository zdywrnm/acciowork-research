# OpenTrad 架构决策一页纸

抓取日期：2026-04-23（America/Los_Angeles）  
一句话：OpenTrad v1 应做“本机 Claude Code CLI 的行业化 GUI”，不是重做 Agent runtime。

## 必须拍板的架构判断

1. **主集成路径：本地 Claude Code CLI subprocess/PTY。** 用 `claude -p --output-format stream-json --verbose --mcp-config <generated>` 拿结构化事件；登录/权限/异常时 fallback 到 PTY。不要把 Claude Agent SDK 作为默认主路径，因为官方 SDK 文档要求第三方产品使用 API key/云厂商凭证，不能默认复用 claude.ai 订阅额度。
2. **认证策略：不代管 token。** 安装向导只检测 `claude --version`、调用 `claude auth status` 展示脱敏状态；未登录时引导用户跑 `claude auth login --claudeai`。macOS 凭证在 Keychain，OpenTrad 不读取。
3. **MCP 策略：运行时生成 config。** v1 每次任务传 `--mcp-config /path/to/opentrad.mcp.json`，必要时加 `--strict-mcp-config`。项目根 `.mcp.json` 适合高级用户，默认不污染。
4. **会话策略：OpenTrad 管 metadata，Claude 管 transcript。** 用 `--session-id` 创建任务，`--resume`/`--continue` 续聊；本机 transcript 在 `~/.claude/projects/<encoded-cwd>/<session-id>.jsonl`。OpenTrad 只保存标题、skill、cwd、session id、风险状态、摘要。
5. **权限策略：行业风险优先。** Claude Code 有 permission modes，但 OpenTrad 还要有业务级 stop-before-submit：RFQ、发品、OAuth Allow、真实发信、付款都必须停在最终确认前。

## 建议 MVP 技术形态

最快 MVP：**Electron + TypeScript + React + node-pty + child_process**。理由是团队偏 TS/Node，PTY 和 xterm 生态成熟，能最快封装真实 CLI。

更轻方案：**Tauri + React + Rust process/PTY sidecar**。体积更小，但 PTY、更新、签名和跨平台调试成本更高。若总工优先工程速度，先 Electron；若优先轻量发行，再选 Tauri。

Claude Code 本身不要求我们自带 Node：官方 setup 文档说明 npm 包也安装 native binary，安装后的 `claude` 不调用 Node。v1 应优先检测/引导安装官方 Claude Code，而不是把它重新打包进 OpenTrad。

## v1 功能边界

- P0：Claude Code 检测/登录引导、项目选择、session list、stream-json 渲染、MCP config writer、skill launcher、风险确认弹窗、导出 markdown/CSV。
- P0 skills：`trade-email-writer`、`supplier-rfq-draft`、`product-sourcing-1688`，随后补 `listing-localizer-seo`、`hs-code-compliance-check`。
- P1：多 session 并行、worktree isolation、连接器 OAuth、Shopify/TikTok/Amazon 半自动流程、成本/额度更完整看板。
- 不做：自研模型网关、多用户服务器、自动发布/自动发送、代管 Claude 凭证、绕过 Claude Code 权限系统。

## 关键风险

- `stream-json` 是 CLI 输出协议，不是长期稳定 SDK；需要按 Claude Code version 做兼容层。
- 订阅复用的合规边界要清楚：我们只是启动用户本机 CLI，不代理、不转售、不读取 token。
- 非交互模式遇到权限 prompt 可能卡住；必须设计 PTY fallback 或 MCP permission prompt tool。
- MCP 工具过多会污染上下文和提高误调用风险；每个 skill 必须最小工具集。
- 外贸动作副作用大，OpenTrad 的“最终确认前停止”要比 Claude Code 默认权限更严格。

## 可偷设计

- Cline：diff/command 人工批准、checkpoint、成本透明。
- Zed：thread sidebar、Write/Ask/Minimal profiles、worktree isolation、ACP 作为未来外部 agent 协议。
- LibreChat/Open WebUI：provider/endpoint 配置和 agent schema，但 v1 先不做多模型。
- Cherry Studio：中文桌面客户端体验、模型配置 UI、MCP 管理页。

## 下一步给总工

先写 OpenTrad v1 架构文档时，以“Process Manager + Stream Parser + Skill Manifest + MCP Config Writer + Risk Gate”五个模块拆分。不要从大而全 agent 平台开始；v1 的护城河是外贸 skill 和安全确认，不是模型 runtime。
