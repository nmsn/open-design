# Open Design — 代码库完整概览

> 自动生成于 2026-06-25，覆盖 v0.11.1 版本。

---

## 1. 项目标识

| | |
|---|---|
| **名称** | `open-design` |
| **版本** | 0.11.1 |
| **许可证** | Apache-2.0 |
| **描述** | 本地优先、开源的 Claude Design 替代方案 — 一个检测已安装编码代理 CLI、运行设计技能 + 设计系统、并将制品流式传输到沙箱预览的设计工作区。 |
| **包管理器** | pnpm 10.33.2（通过 Corepack） |
| **运行时目标** | Node ~24 |
| **模块系统** | ESM（全局 `"type": "module"`） |

---

## 2. 技术栈

### 编程语言

| 语言 | 用途 |
|---|---|
| **TypeScript** | 主语言（~100% 源代码） |
| **JavaScript** | 仅用于生成/供应商输出、bin shim |
| **CSS** | Tailwind CSS 4.3.0（通过 PostCSS）+ CSS Modules |

### 框架与库

| 层级 | 技术栈 |
|---|---|
| 前端 | **Next.js 16**（App Router）+ **React 18** |
| 样式 | **Tailwind CSS 4.3** + CSS Modules |
| 动画 | **Motion**（Framer Motion 继任者）12.40 |
| 富文本 | **Lexical** 0.36.2 |
| 语法高亮 | **Shiki** 4.1 |
| 终端 | **xterm.js** 5.5 |
| 后端 | **Express 5.2**（守护进程 HTTP 服务器） |
| 数据库 | **better-sqlite3** 12.10（SQLite, WAL 模式） |
| 流式传输 | **Server-Sent Events**（SSE）用于代理输出 |
| 桌面端 | **Electron** 41.3 |
| 构建工具 | **esbuild** 0.28（包）, **Next.js**（web）, **tsc**（类型检查）, **electron-builder** 26.8（打包） |
| 测试 | **Vitest** 4.1.6（单元/集成）, **Playwright** 1.60（E2E/UI）, **pixelmatch**（视觉回归） |
| MCP | **@modelcontextprotocol/sdk** 1.29 |
| AI SDK | **@anthropic-ai/sdk** 0.32, **openai** 6.38 |
| 遥测 | **PostHog**（浏览器 + node）, **OpenTelemetry** |
| 监控 | **prom-client**（Prometheus 指标） |
| 可观测性 | **Langfuse**（追踪桥接） |

### 核心依赖

| 依赖 | 用途 |
|---|---|
| `zod` 3.25 | 模式验证 |
| `jszip` | ZIP 导出 |
| `multer` 2.1 | 文件上传 |
| `cheerio` 1.2 | HTML 解析 |
| `chokidar` 5.0 | 文件监听 |
| `node-pty` 1.1 | 终端模拟 |
| `tar` 7.5 | 归档处理 |
| `blake3-wasm` 2.1 | 内容哈希 |
| `@powerformer/vela-cli` | 可选的 AMR CLI |

---

## 3. 根目录配置文件

| 文件 | 用途 |
|---|---|
| `package.json` | 根工作区包，定义 `od` 入口 `apps/daemon/bin/od.mjs`，脚本（`guard`、`typecheck`、`tools-dev`、`tools-pack` 等） |
| `pnpm-workspace.yaml` | 工作区包：`packages/*`、`apps/*`、`tools/*`、`e2e` |
| `.node-version` | 包含 `24` |
| `vercel.json` | Vercel 部署配置：构建 `@open-design/web`，输出到 `apps/web/out` |
| `flake.nix` | Nix flake，用于可复现的开发环境 |
| `mise.toml` | Mise 工具版本管理器配置 |
| `.gitignore` | 标准忽略规则，包括 `.tmp/`、`.next/`、`dist/` |
| `.dockerignore` | Docker 构建排除项 |
| `AGENTS.md` | AI 代理的主目录指南 — 仓库规范的唯一权威来源 |
| `CLAUDE.md` | 最小化 Claude Code 配置 |
| `CONTEXT.md` | 领域术语表，定义 Project、Artifact、Live Artifact 等概念 |

> **注意：** 根目录没有 `tsconfig.json`，每个包/应用都有自己的。

---

## 4. 目录结构

### 4.1 `apps/` — 应用包（6 个应用）

| 目录 | 包名 | 角色 |
|---|---|---|
| `apps/web` | `@open-design/web` | Next.js 16 App Router + React 18 前端 |
| `apps/daemon` | `@open-design/daemon` | 本地特权守护进程（Express + SQLite），即 `od` CLI |
| `apps/desktop` | `@open-design/desktop` | Electron 外壳（Electron 41.3） |
| `apps/packaged` | `@open-design/packaged` | 打包的 Electron 运行时入口，esbuild 打包 |
| `apps/landing-page` | — | 营销落地页 |
| `apps/telemetry-worker` | — | 遥测处理工作进程 |

### 4.2 `packages/` — 共享库（15 个包）

| 包 | 用途 |
|---|---|
| `@open-design/contracts` | 纯 TypeScript API 契约（DTO、SSE 事件、错误形状），web 和 daemon 共享。使用 Zod 进行验证。 |
| `@open-design/components` | 共享 React UI 原语（Button、Dialog、Input、Select、Textarea、VisuallyHidden） |
| `@open-design/sidecar` | 通用 sidecar 运行时 — IPC 服务器/客户端（Unix 套接字 / Windows 命名管道）、端口分配、命名空间解析、运行时路径管理 |
| `@open-design/sidecar-proto` | Open Design sidecar 业务协议定义 |
| `@open-design/platform` | 操作系统进程原语 — spawn、kill、代理检测、进程戳匹配、用户工具链 bin 发现 |
| `@open-design/host` | 渲染器宿主桥接协议 |
| `@open-design/launcher-proto` | 启动器协议定义 |
| `@open-design/release` | 发布领域原语 |
| `@open-design/diagnostics` | 日志收集、脱敏、zip 打包用于诊断导出 |
| `@open-design/download` | 下载处理 |
| `@open-design/agui-adapter` | OD 事件与 AG-UI 协议（CopilotKit）之间的双向适配器 |
| `@open-design/plugin-runtime` | 插件清单解析、验证、合并、引用解析（纯 TS，无 node:fs） |
| `@open-design/registry-protocol` | 插件注册表后端协议 |
| `@open-design/metatool` | 内部元数据辅助工具，用于构建新鲜度检查 |

### 4.3 `tools/` — 构建/生命周期工具（4 个工具）

| 工具 | 包名 | 用途 |
|---|---|---|
| `tools/dev` | `@open-design/tools-dev` | 本地开发生命周期：`start`/`stop`/`run`/`status`/`logs`/`inspect`/`check`。入口：`pnpm tools-dev`。 |
| `tools/pack` | `@open-design/tools-pack` | Electron 打包构建：electron-builder、公证、安装器身份验证、beta 发布准备 |
| `tools/serve` | `@open-design/tools-serve` | 固件服务：确定性的更新器元数据和制品 |
| `tools/release` | `@open-design/tools-release` | 发布自动化 |

### 4.4 `e2e/` — 端到端测试

- **Playwright** 用于 UI 自动化，**Vitest** 用于 API/规范测试
- 优先级：P0、P1、P2
- 测试目录：`tests/`、`specs/`、`ui/`
- 支持视觉回归测试（`playwright.visual.config.ts`）
- 资源在 `resources/`，共享库在 `lib/`

### 4.5 `skills/` — 代理设计技能（~160 个文件夹）

- 每个技能是一个包含 `SKILL.md` + 支持文件的文件夹
- 遵循 Claude Code SKILL.md 约定，带有 `od:` frontmatter 扩展
- **模式：** `prototype`、`deck`、`image`、`video`、`audio`、`template`、`design-system`、`utility`
- **场景：** `design`、`marketing`、`operation`、`engineering`、`product`、`finance`、`hr`、`sale`、`personal`

### 4.6 `design-templates/` — 渲染目录（~113 个模板）

- 幻灯片模板：`html-ppt-*`（15 个模板 × 36 个主题）、`guizang-ppt`
- 原型模板：`web-prototype`、`mobile-app`、`dashboard`、`saas-landing` 等
- 视频：`hyperframes/`（HTML 转 MP4 动态图形）
- 图片/视频/音频模板

### 4.7 `design-systems/` — 品牌 DESIGN.md 系统（~152 个品牌）

- 每个是一个包含 `DESIGN.md` 文件的文件夹
- **9 段式模式：** 颜色、排版、间距、布局、组件、动效、语调、品牌、反模式
- 覆盖 AI、开发者工具、金融科技、电商、媒体、汽车等行业

### 4.8 `craft/` — 通用品牌无关的工艺规则

- Markdown 文件覆盖：无障碍基线、动画纪律、反 AI 味道、颜色、表单验证、UX 定律、RTL/双向文本、状态覆盖、排版层级
- 技能通过 `od.craft.requires` 选择性启用

### 4.9 `mocks/` — 基于回放的模拟 CLI

- 模拟代理：opencode、claude、codex、gemini、cursor-agent、deepseek、qwen、grok、devin、hermes、kilo、kimi、kiro、vela、vela (AMR)
- 基于匿名化的 Langfuse 追踪构建
- 用于测试和自验证
- 包含 `manifest.json`、`mock-agent.mjs`、`golden/` 固件

### 4.10 `plugins/` — 插件生态系统

- `_official/`：261 个官方插件（场景、图片模板、视频模板、设计系统、原子、示例）
- `community/`：社区插件
- `registry/`：发布流程
- `spec/`：插件规范（SPEC.md、AGENT-DEVELOPMENT.md、CONTRIBUTING.md、PUBLISHING-REGISTRIES.md）

### 4.11 其他目录

| 目录 | 用途 |
|---|---|
| `docs/` | 架构、协议、代理适配器、模式、国际化、部署指南 |
| `deploy/` | Docker/Docker Compose、AWS、Azure 部署配置 |
| `scripts/` | 构建/守护/种子/同步脚本 |
| `prompt-templates/` | 图片/视频提示词模板 |
| `data/`、`specs/`、`templates/`、`story/`、`assets/`、`nix/`、`clipper/`、`figma-plugin/` | 各种辅助目录 |

---

## 5. 数据库表结构（SQLite）

守护进程使用单个 SQLite 数据库（`app.sqlite`），采用 WAL 模式。核心表：

| 表 | 列 | 用途 |
|---|---|---|
| `projects` | id, name, skill_id, design_system_id, pending_prompt, metadata_json, timestamps | 项目注册表 |
| `templates` | id, name, description, source_project_id, files_json | 可复用模板 |
| `conversations` | id, project_id, title, session_mode, timestamps | 聊天会话（外键 → projects） |
| `agent_sessions` | conversation_id, agent_id, session_id, stable_prompt_hash | 代理会话追踪 |
| `messages` | id, conversation_id, role, content, agent_id, events_json, attachments_json, produced_files_json, feedback_json, run_context_json, applied_plugin_snapshot_json, timestamps | 聊天消息 |
| `preview_comments` | id, project_id, conversation_id, file_path, element_id, selector, label, note, status, slide_index, timestamps | 制品评审评论 |
| `tabs` | project_id | 工作区标签页 |

还有其他表用于评审、媒体任务、库和插件（通过 `apps/daemon/src/db.ts` 中的迁移函数管理）。

---

## 6. API 路由

守护进程在 `localhost:7456` 上暴露 `/api/*` 下的 HTTP 路由。核心路由：

| 路由 | 方法 | 用途 |
|---|---|---|
| `/api/health` | GET | 健康检查 |
| `/api/agents` | GET | 列出检测到的编码代理 CLI |
| `/api/skills` | GET | 列出可用技能 |
| `/api/design-systems` | GET | 列出设计系统 |
| `/api/projects` | GET/POST | 项目 CRUD |
| `/api/import/folder` | POST | 导入现有文件夹为项目 |
| `/api/import/claude-design` | POST | 导入 Claude Design ZIP |
| `/api/projects/:id/files` | GET | 项目文件列表 |
| `/api/projects/:id/upload` | POST | 文件上传 |
| `/api/chat` | POST | SSE 流式聊天（代理输出） |
| `/api/proxy/{provider}/stream` | POST | BYOK 代理（SSRF 防护），支持 anthropic、openai、azure、google、ollama、senseaudio |
| `/api/artifacts/save` | POST | 保存制品 |
| `/api/artifacts/lint` | POST | 制品 lint 检查 |
| `/api/plugins` | GET | 插件市场 |
| `/api/routines` | GET | 自动化例程 |
| `/api/version` | GET | 版本信息 |

**路由文件**位于 `apps/daemon/src/routes/`：
`active-context.ts`、`automation.ts`、`chat.ts`、`daemon.ts`、`deploy.ts`、`design-system-tool.ts`、`design-systems.ts`、`genui.ts`、`handoff.ts`、`host-tools.ts`、`library.ts`、`live-artifact.ts`、`media.ts`、`memory.ts`、`plugins/`、`project/`、`routine.ts`、`runs.ts`、`social-share.ts`、`static-resource.ts`、`telemetry.ts`、`terminal.ts`、`vela.ts`、`xai.ts`

---

## 7. 代理适配器系统

守护进程通过 `apps/daemon/src/runtimes/` 中的适配器支持 **22+ 编码代理 CLI**：

| 模块 | 用途 |
|---|---|
| `detection.ts`、`resolution.ts` | PATH 扫描 + 配置目录探测，发现已安装的代理 |
| `registry.ts` | 代理定义、配置文件管理 |
| `launch.ts`、`env.ts` | 使用标准化环境变量生成代理进程 |
| `claude-stream.ts`、`json-event-stream.ts`、`qoder-stream.ts` | 将代理 stdout 解析为结构化事件 |
| `capabilities.ts` | 每个代理的功能矩阵 |
| `mcp.ts` | 实时制品 MCP 服务器配置 |
| `models.ts` | 模型注册表、AMR 集成 |

### 支持的代理

Claude Code、Codex、Cursor、Copilot、OpenClaw、Antigravity、Gemini CLI、OpenCode、Qwen、Qoder、Hermes、Kimi、Pi Agent、Mistral Vibe、Cline、Trae、DeepSeek、Kiro、Kilo、Devin

---

## 8. 入口点

| 入口 | 文件 | 用途 |
|---|---|---|
| `od` CLI | `apps/daemon/bin/od.mjs` | 主 CLI 二进制文件 |
| 守护进程服务器 | `apps/daemon/src/server.ts` | Express 服务器，所有 `/api/*` 路由 |
| CLI 命令 | `apps/daemon/src/cli.ts` | 所有 `od` 子命令 |
| Web 应用 | `apps/web/app/layout.tsx` + `app/[[...slug]]/` | Next.js App Router 入口 |
| 主 React 应用 | `apps/web/src/App.tsx` | 主 React 应用组件 |
| 桌面端主进程 | `apps/desktop/src/main/index.ts` | Electron 主进程 |
| 打包入口 | `apps/packaged/dist/index.mjs` | esbuild 打包的运行时 |
| tools-dev | `tools/dev/bin/tools-dev.mjs` | 开发生命周期 CLI |
| tools-pack | `tools/pack/bin/tools-pack.mjs` | 构建/打包 CLI |

---

## 9. 核心源码目录 — 详解

### 守护进程（`apps/daemon/src/`）— ~140 个文件

| 子目录 | 用途 |
|---|---|
| `server.ts`、`cli.ts` | 主服务器和 CLI（各 ~9K 行） |
| `db.ts` | SQLite 表结构和查询 |
| `runtimes/` | 30+ 文件：代理检测、生成、流式传输 |
| `routes/` | 25+ 路由模块 |
| `plugins/` | 40+ 插件管理文件 |
| `critique/` | 18 个文件：制品自评审系统 |
| `artifacts/` | 制品处理 |
| `design-systems/` | 设计系统管理 |
| `integrations/` | 外部集成 |
| `media/`、`media-adapters/` | 媒体生成 |
| `prompts/` | 系统提示词组合 |
| `memory.ts` | 项目记忆/上下文 |

### Web（`apps/web/src/`）— ~27 个子目录

| 子目录 | 用途 |
|---|---|
| `App.tsx` | 主应用组件 |
| `components/` | 180+ React 组件 |
| `runtime/` | 代理运行时集成 |
| `state/` | 应用状态管理 |
| `providers/` | React 上下文提供者 |
| `i18n/` | 国际化（24 个语言文件） |
| `artifacts/` | 制品展示 |
| `analytics/` | 分析集成 |
| `edit-mode/` | 就地编辑 |
| `hooks/`、`lib/`、`utils/` | 共享工具 |

---

## 10. 构建与开发工作流

| 命令 | 用途 |
|---|---|
| `pnpm tools-dev run web` | 启动开发环境（Next.js + 守护进程） |
| `pnpm guard` | 风格策略、产品中立性、导入隔离检查 |
| `pnpm typecheck` | 所有包的类型检查 |
| `pnpm --filter @open-design/web build` | 构建 web 应用 |
| `pnpm --filter @open-design/daemon build` | 构建守护进程 |
| `pnpm --filter @open-design/desktop build` | 构建桌面端 |
| `pnpm --filter @open-design/web test` | 运行 web 测试 |
| `pnpm --filter @open-design/daemon test` | 运行守护进程测试 |
| `pnpm tools-pack mac build --to all` | 构建 macOS 包 |
| `pnpm tools-pack win build --to nsis` | 构建 Windows 包 |
| `pnpm tools-pack linux build --to appimage` | 构建 Linux 包 |
