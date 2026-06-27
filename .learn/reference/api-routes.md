# Open Design API 路由参考

> 基于 `apps/daemon/src/server.ts` · 共 42 个路由模块 · 守护进程默认端口 7456

---

## 路由注册总览

所有路由通过 `registerXxxRoutes(app, ctx)` 模式注册，定义在 `apps/daemon/src/routes/` 目录下。

---

## 🗣️ 聊天与运行

| 路由模块 | 源文件 | 挂载路径 | 用途 |
|---------|--------|---------|------|
| `registerChatRoutes` | `routes/chat.ts` | `/api/chat`, `/api/proxy/*` | 聊天 SSE 流、BYOK AI 代理 |
| `registerRunRoutes` | `routes/runs.ts` | `/api/runs` | 代理运行管理（启动、停止、状态） |
| `registerTerminalRoutes` | `routes/terminal.ts` | `/api/terminal` | 终端会话（SSE 流式） |

## 📁 项目与文件

| 路由模块 | 源文件 | 挂载路径 | 用途 |
|---------|--------|---------|------|
| `registerProjectRoutes` | `routes/project/index.ts` | `/api/projects` | 项目 CRUD（33 个子路由） |
| `registerProjectFileRoutes` | `routes/project/index.ts` | `/api/projects/:id/files` | 文件读写、搜索、文件夹管理 |
| `registerProjectUploadRoutes` | `routes/project/index.ts` | `/api/upload` | 文件上传（最多 8 张图片） |
| `registerProjectExportRoutes` | `import-export-routes.ts` | `/api/projects/:id/export` | 项目导出 |
| `registerProjectArtifactRoutes` | `routes/project/index.ts` | `/api/artifacts` | 制品保存、lint 检查 |
| `registerImportRoutes` | `import-export-routes.ts` | `/api/import` | 导入（文件夹、Claude Design ZIP） |
| `registerFinalizeRoutes` | `import-export-routes.ts` | — | 完成处理 |
| `registerHandoffRoutes` | `routes/handoff.ts` | `/api/handoff` | 交接协议 |

## 🎨 设计系统与品牌

| 路由模块 | 源文件 | 挂载路径 | 用途 |
|---------|--------|---------|------|
| `registerDesignSystemRoutes` | `routes/design-systems.ts` | `/api/design-systems` | 设计系统 CRUD |
| `registerDesignSystemToolRoutes` | `routes/design-system-tool.ts` | `/api/design-system-tool` | 设计系统工具调用 |
| `registerBrandRoutes` | `brand-routes.ts` | `/api/brands` | 品牌提取（从 HTML 提取设计规范） |

## 🔌 插件系统

| 路由模块 | 源文件 | 挂载路径 | 用途 |
|---------|--------|---------|------|
| `registerPluginRoutes` | `routes/plugins/index.ts` | `/api/plugins` | 插件管理 |
| `registerPluginEventRoutes` | `routes/plugins/index.ts` | `/api/plugins/events` | 插件事件订阅 |
| `registerPluginMarketplaceRoutes` | `routes/plugins/marketplaces.ts` | `/api/plugins/marketplace` | 插件市场浏览 |
| `registerPluginAssetRoutes` | `routes/plugins/assets.ts` | `/api/plugins/assets` | 插件静态资源 |
| `registerProjectPluginRoutes` | `routes/plugins/index.ts` | `/api/projects/:id/plugins` | 项目级插件绑定 |

## 🤖 代理与 AI

| 路由模块 | 源文件 | 挂载路径 | 用途 |
|---------|--------|---------|------|
| `registerMcpRoutes` | `mcp-routes.ts` | `/api/mcp` | MCP 服务器配置 |
| `registerXaiRoutes` | `routes/xai.ts` | `/api/xai` | XAI（可解释 AI）相关 |
| `registerGenuiRoutes` | `routes/genui.ts` | `/api/genui` | 生成式 UI |
| `registerLiveArtifactRoutes` | `routes/live-artifact.ts` | `/api/live-artifacts` | 实时制品预览 |
| `registerAtomRoutes` | `routes/` | `/api/atoms` | 原子组件 |

## 🚀 部署与媒体

| 路由模块 | 源文件 | 挂载路径 | 用途 |
|---------|--------|---------|------|
| `registerDeployRoutes` | `routes/deploy.ts` | `/api/deploy` | 部署管理 |
| `registerDeploymentCheckRoutes` | `routes/deploy.ts` | `/api/deploy/check` | 部署状态检查 |
| `registerMediaRoutes` | `routes/media.ts` | `/api/media` | 媒体生成（图片/视频/音频） |
| `registerSocialShareRoutes` | `routes/social-share.ts` | `/api/social-share` | 社交分享链接生成 |
| `registerVelaRoutes` | `routes/vela.ts` | `/api/vela` | Vela CLI 集成（AMR） |

## ⚙️ 系统与基础设施

| 路由模块 | 源文件 | 挂载路径 | 用途 |
|---------|--------|---------|------|
| `registerDaemonRoutes` | `routes/daemon.ts` | `/api/daemon` | 守护进程状态、关机 |
| `registerMemoryRoutes` | `routes/memory.ts` | `/api/memory` | 项目记忆/上下文管理 |
| `registerAutomationRoutes` | `routes/automation.ts` | `/api/automation` | 自动化例程 |
| `registerRoutineRoutes` | `routes/routine.ts` | `/api/routines` | 定时任务 CRUD |
| `registerTelemetryRoutes` | `routes/telemetry.ts` | `/api/telemetry` | 遥测数据上报 |
| `registerConnectorRoutes` | `connectors/routes.ts` | `/api/connectors` | 外部服务连接器 |
| `registerActiveContextRoutes` | `routes/active-context.ts` | `/api/active-context` | 活跃上下文管理 |
| `registerHostToolsRoutes` | `routes/host-tools.ts` | `/api/host-tools` | 宿主工具（桌面端调用） |

## 📚 库与资源

| 路由模块 | 源文件 | 挂载路径 | 用途 |
|---------|--------|---------|------|
| `registerLibraryRoutes` | `routes/library.ts` | `/api/library` | 素材库管理 |
| `registerStaticResourceRoutes` | `routes/` | `/api/static` | 静态资源服务 |
| `registerOpenDesignPublicMetadataRoutes` | `routes/open-design-public-metadata.ts` | `/api/open-design` | 公开元数据 |

---

## 快速查找

### 按源文件查找

| 源文件 | 路由模块数 |
|--------|-----------|
| `routes/project/index.ts` | 5 |
| `routes/chat.ts` | 1 |
| `routes/plugins/index.ts` | 3 |
| `routes/deploy.ts` | 2 |
| `routes/runs.ts` | 1 |
| `routes/media.ts` | 1 |
| `routes/design-systems.ts` | 1 |
| `routes/routine.ts` | 1 |
| `routes/terminal.ts` | 1 |

### 按 HTTP 方法统计（project/index.ts 内）

| 方法 | 数量 |
|------|------|
| GET | 18 |
| POST | 9 |
| PUT | 2 |
| PATCH | 1 |
| DELETE | 6 |

---

## 直接定义在 server.ts 的路由

除了模块化路由外，server.ts 还直接定义了 3 个轻量路由：

| 方法 | 路由 | 用途 |
|------|------|------|
| GET | `/api/health` | 健康检查 |
| GET | `/api/ready` | 就绪检查 |
| GET | `/api/version` | 版本信息 |

---

## 中间件挂载

| 中间件 | 用途 |
|--------|------|
| `express.json({ limit: '128mb' })` | `/api/library/ingest` 大文件上传 |
| `express.json({ limit: '32mb' })` | `/api/brands/:id/extract-from-html` |
| `express.json({ limit: '4mb' })` | 默认 JSON 解析 |
| `express.static(STATIC_DIR)` | 静态文件服务 |
| `express.static(ARTIFACTS_DIR)` | `/artifacts` 制品静态服务 |
| `express.static(FRAMES_DIR)` | `/frames` 帧静态服务 |
