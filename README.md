# Accio Work Research

## 项目说明

本仓库用于记录对 Accio Work（公开商业软件客户端）的技术调研，目标是为开源对标工具的架构设计、能力拆解和实现规划提供证据化输入。

## 合规声明

所有调研行为仅限于研究者本人设备、本人合法账号 session 与本机产生的数据范围内完成。

- 不涉及反编译加密代码
- 不涉及他人数据
- 不涉及绕过任何许可校验、DRM 或鉴权机制
- 抓包与日志样本仅保留脱敏后的公开版本，统一使用 `<REDACTED>` 占位

## 调研对象版本

- 产品：Accio
- 版本：v0.7.1
- 首次抓取日期：2026-04-22
- 本仓库整理日期：2026-04-23

## 目录结构索引

```text
.
├── .gitignore
├── 00-summary.md
├── 01-tech-stack.md
├── 02-agent-skill-model.md
├── 03-task-walkthrough.md
├── 04-pain-points.md
├── 05-ecosystem.md
├── README.md
├── 06-screenshots/
│   ├── p0-finder-contents-cropped.png
│   ├── p0-finder-contents.png
│   ├── p0-finder-resources-cropped.png
│   └── p0-finder-resources.png
├── 07-raw-evidence/
│   ├── p0-asar-key-paths.txt
│   ├── p0-bundle-structure.txt
│   ├── p0-electron-framework-info.txt
│   ├── p0-frameworks.txt
│   ├── p0-info-plist.txt
│   ├── p0-lib-fingerprints.txt
│   ├── p0-mdls.txt
│   ├── p0-native-modules.txt
│   ├── p0-package.json
│   ├── p0-renderer-index.html
│   ├── p0-renderer-react-signals.txt
│   ├── p0-resources.txt
│   ├── p0-ui-state-fingerprints.txt
│   ├── p0-unpacked-node-modules.txt
│   ├── p02-agent-core-sample.md
│   ├── p02-agent-profile-sample.redacted.jsonc
│   ├── p02-main-exports.txt
│   ├── p02-out-main-index.js
│   ├── p02-phoenix-package-jsons.txt
│   ├── p02-phoenix-package-paths.txt
│   ├── p02-phoenix-packages.txt
│   ├── p02-plugin-agent-template.json
│   ├── p02-runtime-snippets.md
│   ├── p02-session-mechanism-clues.txt
│   ├── p02-session-topology.redacted.md
│   ├── p02-skill-sample-1688-sourcing.md
│   ├── p02-vector-memory-status.txt
│   ├── p02-vector-schema.txt
│   ├── p03-browser-relay-handshake.md
│   ├── p03-model-list-full.json
│   ├── p03-model-list.json
│   ├── p03-model-memory-clues.txt
│   └── p03-routing-sample.redacted.json
└── 08-operation-log.md
```

## 公开仓库说明

仓库内只保留可公开的脱敏证据、总结文档与操作日志；若后续产生原始抓包、Cookie、未脱敏请求体或含真实凭证的临时文件，将只保留在本地并加入 `.gitignore`。
