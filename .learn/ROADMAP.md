# 学习路线图：从前端到全栈 AI 应用工程师

> 以 Open Design 项目为主线，共 4 个阶段，约 20 节课。

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

## 阶段四：进阶专题（第 16-20 课）

> 目标：理解高级架构决策，能做技术判断。

| # | 课程 | 核心概念 | 对应源码 |
|---|------|---------|---------|
| 16 | Electron 桌面端 | 主进程/渲染进程、IPC、Sidecar | `apps/desktop/`, `packages/sidecar/` |
| 17 | 设计系统引擎 | DESIGN.md 解析、9 段式模式 | `design-systems/`, `routes/design-systems.ts` |
| 18 | 制品自评审系统 | Critique 引擎、反馈循环 | `critique/` |
| 19 | 多代理编排 | 并发控制、错误恢复 | `runtimes/` (advanced) |
| 20 | 架构决策复盘 | Trade-off 分析、系统设计思维 | `docs/architecture.md` |

---

## 如何使用

1. 按顺序完成课程，每课 15-30 分钟
2. 完成每课的"检查理解"和"动手探索"
3. 遇到不懂的，用"向 AI 老师提问"框提问
4. 每完成 5 课，尝试为项目提一个小 PR
