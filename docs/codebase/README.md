# Open Design — 代码库文档

> 自动生成于 2026-06-25，覆盖 v0.11.1 版本。

## 什么是 Open Design？

Open Design 是一个**本地优先、开源的设计工作区** — Claude Design 的替代方案。它能检测用户已安装的编码代理 CLI，运行设计技能 + 设计系统，并将生成的制品流式传输到沙箱预览中。

它是**代理原生、模型无关的**：自身不附带 AI 代理，而是发现用户 PATH 中已有的代理 CLI（Claude Code、Codex、Cursor、Gemini CLI 等），通过统一的技能驱动协议进行编排。

## 文档索引

| 文档 | 描述 |
|---|---|
| [OVERVIEW.md](./OVERVIEW.md) | 完整项目概览 — 技术栈、目录结构、数据库表结构、API 路由、代理适配器、入口点 |
| [ARCHITECTURE.md](./ARCHITECTURE.md) | 架构图、数据流、部署拓扑、设计决策 |

## 快速链接

- **根目录 AGENTS.md**：`../../AGENTS.md` — 仓库规范的唯一权威来源
- **架构文档**：`../architecture.md`、`../spec.md`、`../modes.md`
- **插件规范**：`../../plugins/spec/SPEC.md`
- **贡献指南**：`../../CONTRIBUTING.md`

## 速览

| | |
|---|---|
| **版本** | 0.11.1 |
| **许可证** | Apache-2.0 |
| **语言** | TypeScript (ESM) |
| **运行时** | Node ~24 |
| **包管理器** | pnpm 10.33.2 (Corepack) |
| **前端** | Next.js 16 + React 18 + Tailwind CSS 4.3 |
| **后端** | Express 5.2 + SQLite (better-sqlite3) |
| **桌面端** | Electron 41.3 |
| **测试** | Vitest 4.1.6 + Playwright 1.60 |
| **CI/CD** | GitHub Actions, Vercel (web), electron-builder (desktop) |
