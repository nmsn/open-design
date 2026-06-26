# 学习路线图：从前端到全栈 AI 应用工程师

> 以 Open Design 项目为主线，共 5 个阶段，约 25 节课。

---

## 阶段一：后端基础（第 1-5 课）

> 目标：理解 Node.js 后端的核心概念，能读懂 daemon 源码。

| # | 课程 | 核心概念 | 对应源码 |
|---|------|---------|---------|
| 1 | ✅ 项目全景与全栈心智模型 | 架构总览、前后端映射 | `server.ts`, `db.ts` |
| 2 | Express 路由与中间件 | Router、中间件链、错误处理 | `routes/*.ts` |
| 3 | SQLite 数据库操作 | CRUD、事务、迁移 | `db.ts` |
| 4 | 子进程与进程管理 | child_process、spawn、stdin/stdout | `runtimes/launch.ts` |
| 5 | 环境搭建与本地运行 | tools-dev、端口、命名空间 | `tools/dev/` |

## 阶段二：AI 应用核心（第 6-10 课）

> 目标：理解 AI 应用的关键技术模式。

| # | 课程 | 核心概念 | 对应源码 |
|---|------|---------|---------|
| 6 | SSE 流式通信 | Server-Sent Events、事件流解析 | `routes/chat.ts` |
| 7 | Agent 适配器模式 | 检测、注册、启动、流解析 | `runtimes/` |
| 8 | 系统提示词组合 | 上下文注入、技能 + 设计系统 | `prompts/system.ts` |
| 9 | MCP 协议入门 | 工具定义、stdio 通信 | `runtimes/mcp.ts` |
| 10 | BYOK 代理与安全 | SSRF 防护、API 代理 | `routes/chat.ts` (proxy) |

## 阶段三：工程实践（第 11-15 课）

> 目标：能为项目贡献代码。

| # | 课程 | 核心概念 | 对应源码 |
|---|------|---------|---------|
| 11 | Contracts 层设计 | DTO、Zod 验证、类型共享 | `packages/contracts/` |
| 12 | 插件系统架构 | 清单解析、验证、合并 | `plugins/`, `packages/plugin-runtime/` |
| 13 | 测试策略 | Vitest 单元测试、Playwright E2E | `tests/`, `e2e/` |
| 14 | 代码规范与 Guard | pnpm guard、类型检查 | `scripts/guard.ts` |
| 15 | 提交第一个 PR | 分支策略、PR 模板、Code Review | `.github/` |

## 阶段四：Electron 桌面端（第 16-22 课）⭐ 重点

> 目标：理解 Electron 应用的完整生命周期，掌握主进程/渲染进程架构、安全模型、原生集成和自动更新。能独立开发 Electron 桌面应用。
>
> 为什么是重点：Open Design 的 Electron 层是最复杂的部分——7 个包协作、自定义更新系统（3200+ 行）、Sidecar 子进程架构、跨平台原生集成、完整的安全边界设计。这是前端开发者迈向"全栈 + 桌面"的关键跳板。

| # | 课程 | 核心概念 | 对应源码 |
|---|------|---------|---------|
| 16 | Electron 基础：进程模型 | 主进程 vs 渲染进程、BrowserWindow、preload 脚本、contextBridge | `apps/desktop/src/main/index.ts`, `preload.cts` |
| 17 | IPC 通信深潜 | ipcMain.handle / ipcRenderer.invoke、双向通信、类型安全的 Bridge 接口 | `preload.cts`, `runtime.ts`, `packages/host/` |
| 18 | Sidecar 子进程架构 | Unix socket / Windows named pipe、JSON IPC 协议、进程戳（stamp）、守护进程编排 | `packages/sidecar/`, `packages/sidecar-proto/`, `apps/packaged/src/sidecars.ts` |
| 19 | 窗口管理与原生 UI | 多窗口管理、闪屏动画、桌面宠物、菜单系统、PDF/图片导出 | `runtime.ts`, `pdf-export.ts`, `artifact-export.ts` |
| 20 | 安全架构 | contextIsolation、sandbox、nodeIntegration、HMAC 认证、URL 白名单、webview 隔离 | `runtime.ts`, `preload.cts` |
| 21 | 自定义协议与打包 | od:// 协议注册、esbuild 打包、单实例锁、命名空间路径、Headless 模式 | `apps/packaged/src/protocol.ts`, `launch.ts`, `paths.ts` |
| 22 | 自动更新系统 | 元数据拉取、校验和验证、平台特定安装器、延迟安装、指数退避轮询 | `updater.ts`, `packages/launcher-proto/` |

### Electron 学习路径图

```
第 16 课：进程模型（基础）
    │
    ├── 第 17 课：IPC 通信（核心机制）
    │       │
    │       ├── 第 18 课：Sidecar 架构（进阶 IPC）
    │       │
    │       └── 第 19 课：窗口与原生 UI（应用层）
    │
    ├── 第 20 课：安全架构（横切关注点）
    │
    └── 第 21 课：协议与打包（部署）
            │
            └── 第 22 课：自动更新（生产级）
```

## 阶段五：进阶专题（第 23-25 课）

> 目标：理解高级架构决策，能做技术判断。

| # | 课程 | 核心概念 | 对应源码 |
|---|------|---------|---------|
| 23 | 设计系统引擎 | DESIGN.md 解析、9 段式模式 | `design-systems/`, `routes/design-systems.ts` |
| 24 | 制品自评审系统 | Critique 引擎、反馈循环 | `critique/` |
| 25 | 架构决策复盘 | Trade-off 分析、系统设计思维 | `docs/architecture.md` |

---

## 如何使用

1. 按顺序完成课程，每课 15-30 分钟
2. 完成每课的"检查理解"和"动手探索"
3. 遇到不懂的，用"向 AI 老师提问"框提问
4. 每完成 5 课，尝试为项目提一个小 PR
5. 阶段四（Electron）是重点，建议每课花更多时间做动手实验
