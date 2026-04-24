# OpenTrad v1 Skill 首发清单

定位：v1 只包装 Claude Code 成外贸商家能用的桌面 App。Skill 应该是“提示词 + 工具白名单 + 风险边界 + 输出模板”，不要在 v1 做复杂 marketplace。

## P0 推荐首发 5 个

| Skill | 场景 | 调用工具 | 优先级 | 推荐理由 |
| --- | --- | --- | --- | --- |
| `product-sourcing-1688` | 1688/中文货源找品，整理候选商品、价格带、MOQ、供应商风险点 | 浏览器只读、搜索、表格/CSV、本地文件；可选截图 OCR | P0 | 外贸商家高频刚需；主要是只读分析，外部副作用低；能体现“中文供应链 + 英文销售”的差异化 |
| `supplier-rfq-draft` | Alibaba.com / 供应商页面信息整理，生成 RFQ/询盘草稿和问题清单 | 浏览器只读、搜索、本地文档；禁止自动发送 | P0 | 离成交最近，但可停在“发送前确认”；能直接替代 Accio 的跨境工作流卖点 |
| `trade-email-writer` | RFQ、报价、议价、催付、售后、跟进邮件写作 | 本地文件、通讯录导入（只读）、剪贴板/文本输入；禁止真实发信 | P0 | 实现成本最低、价值最直观；不依赖平台登录；适合非技术商家立即上手 |
| `listing-localizer-seo` | 把中文商品资料转成 Amazon/Shopify/TikTok/Alibaba 英文 listing、标题、五点、关键词 | 本地文件、图片读入、搜索；可选平台规则知识库 | P0 | 纯内容生产，风险低；可复用到多平台；是“外贸版 Claude Code”的开箱即用样板 |
| `hs-code-compliance-check` | 初查 HS code、禁限售、认证要求、报关描述和风险提示 | 搜索、官方网页浏览、本地表格；输出必须标注“非法律/报关意见” | P0 | 商家愿意为降风险付费；可做只读研究，不产生副作用；能形成明显行业属性 |

P0 的共同边界：

- 默认不登录用户平台，不发邮件、不发 RFQ、不发布商品。
- 所有“可外发内容”只生成草稿和确认清单。
- 输出统一包含：数据来源、置信度、下一步人工确认项、不可自动执行项。

## P1 建议做

| Skill | 场景 | 工具 | 优先级 |
| --- | --- | --- | --- |
| `competitor-market-research` | 竞品店铺/商品/价格/卖点分析，生成对标表 | 浏览器、搜索、表格 | P1 |
| `shopify-ops-assistant` | Shopify 商品文案、collection 规划、FAQ、政策页草稿 | 浏览器、本地文件；OAuth 到授权页停 | P1 |
| `tiktok-shop-publish-prep` | TikTok Shop 发品素材准备、标题标签、短视频脚本 | 浏览器、本地文件、图片/视频元数据 | P1 |
| `amazon-listing-optimizer` | Amazon listing SEO、review pain point 归纳、A+ 页面草稿 | 浏览器、搜索、表格 | P1 |
| `logistics-quote-compare` | 跨境物流询价条件整理、报价对比、渠道推荐 | 搜索、表格、本地文件 | P1 |
| `supplier-risk-check` | 供应商资质、工商/平台信息、交期和付款风险检查 | 浏览器、搜索、表格 | P1 |

## P2 以后做

| Skill | 场景 | 暂缓原因 |
| --- | --- | --- |
| `auto-rfq-sender` | 自动发送询盘/RFQ | 外部副作用高，需要账号授权和审计 |
| `marketplace-auto-publisher` | 自动发布到 TikTok/Amazon/Shopify/Alibaba | 平台规则复杂，误操作代价高 |
| `crm-followup-automation` | 自动跟进客户、WhatsApp/Telegram/邮件群发 | 涉及真实联系人和消息发送 |
| `erp-inventory-sync` | ERP/库存/订单同步 | 连接器和权限模型复杂 |
| `customs-doc-generator` | 商业发票、装箱单、报关材料自动生成 | 法务/合规风险高，需要模板和人工复核 |

## v1 只选 3-5 个的原因

推荐先做 5 个 P0，但实现排期可以按 3 个核心 skill 启动：

1. `trade-email-writer`
2. `supplier-rfq-draft`
3. `product-sourcing-1688`

这三个覆盖“沟通、询盘、找货”主链路，外部副作用可控，且不需要复杂 OAuth。`listing-localizer-seo` 和 `hs-code-compliance-check` 紧随其后，分别补上内容生产和合规风险两个高价值面。

## Skill 配置建议

每个 skill 建议包含：

```yaml
id: supplier-rfq-draft
title: Supplier RFQ Draft
risk_level: draft_only
allowed_tools:
  - WebSearch
  - WebFetch
  - Read
  - Write
  - mcp__opentrad_browser__*
disallowed_tools:
  - Bash(curl *)
  - Bash(*send*)
  - mcp__gmail__send*
stop_before:
  - send_rfq
  - submit_form
  - oauth_allow
outputs:
  - supplier_summary
  - rfq_draft
  - missing_questions
  - evidence_links
```

v1 不要把 skill 做成脚本包；先用 Markdown/YAML manifest + prompt template，交给 Claude Code + MCP tools 执行。这样最贴近 Claude Code 的现有能力，也方便商家/社区贡献。
