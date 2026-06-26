# 全栈 AI 应用开发学习资源

## Knowledge

### Node.js 后端

- [Node.js 官方文档](https://nodejs.org/docs/latest/)
  权威参考，用于查询 API 细节。Use for: fs、path、child_process、stream 等核心模块。
- [Express.js 官方指南](https://expressjs.com/en/guide/routing.html)
  路由、中间件、错误处理的基础。Use for: 理解 Open Design daemon 的路由结构。
- [better-sqlite3 文档](https://github.com/WiseLibs/better-sqlite3)
  同步 SQLite 驱动的 API 和性能特性。Use for: 理解 Open Design 的数据库层。

### AI 应用开发

- [Anthropic Claude API 文档](https://docs.anthropic.com/en/api)
  Claude API 的完整参考。Use for: 理解 Open Design 的 BYOK 代理和流式处理。
- [OpenAI API 文档](https://platform.openai.com/docs/api-reference)
  OpenAI API 参考。Use for: 理解多模型支持。
- [MCP 协议规范](https://modelcontextprotocol.io/)
  Model Context Protocol 的官方规范。Use for: 理解 Open Design 的 MCP 服务器集成。
- [Server-Sent Events (SSE) - MDN](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)
  SSE 的浏览器 API 和协议细节。Use for: 理解 Open Design 的流式聊天架构。

### TypeScript 全栈

- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/)
  TypeScript 核心概念。Use for: 类型系统、泛型、装饰器等。
- [Zod 文档](https://zod.dev/)
  运行时类型验证库。Use for: 理解 Open Design 的 contracts 层。

### Electron 桌面端

- [Electron 官方文档](https://www.electronjs.org/docs)
  主进程/渲染进程、IPC 通信。Use for: 理解 Open Design 的桌面端架构。
- [Electron 安全指南](https://www.electronjs.org/docs/latest/tutorial/security)
  contextIsolation、sandbox、nodeIntegration 等安全最佳实践。Use for: 理解 Open Design 的安全架构设计。
- [Electron IPC 教程](https://www.electronjs.org/docs/latest/tutorial/ipc)
  ipcMain.handle / ipcRenderer.invoke 的完整用法。Use for: 理解渲染进程与主进程的通信模式。
- [Electron 打包与分发](https://www.electronjs.org/docs/latest/tutorial/webpack)
  electron-builder、auto-updater。Use for: 理解 Open Design 的打包和更新系统。
- [contextBridge API](https://www.electronjs.org/docs/latest/api/context-bridge)
  预加载脚本安全暴露 API 的标准方式。Use for: 理解 preload.cts 的设计。
- [Electron BrowserWindow API](https://www.electronjs.org/docs/latest/api/browser-window)
  窗口创建和管理的完整参考。Use for: 理解多窗口管理、闪屏、桌面宠物等。

## Wisdom (Communities)

- [Open Design GitHub Discussions](https://github.com/nexu-io/open-design/discussions)
  项目官方讨论区。Use for: 架构疑问、贡献前的讨论。
- [Node.js 中文社区](https://nodejs.cn/)
  中文 Node.js 社区。Use for: Node.js 相关问题求助。
- [V2EX - AI 板块](https://www.v2ex.com/go/ai)
  中文技术社区 AI 讨论。Use for: AI 应用开发经验交流。

## Gaps

- Open Design 内部的 Agent 流解析（claude-stream.ts）缺少外部文档，需要通过源码阅读学习
- 插件系统的 `open-design.json` 规范是项目自定义的，无外部参考
