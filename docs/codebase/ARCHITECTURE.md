# Open Design — 架构文档

> 自动生成于 2026-06-25，覆盖 v0.11.1 版本。

---

## 1. 高层架构

```mermaid
graph TD
    subgraph "用户机器"
        Browser["浏览器"]
        Web["Next.js Web 应用<br/>localhost:3000"]
        Daemon["守护进程 (Express)<br/>localhost:7456"]
        SQLite[("SQLite<br/>app.sqlite")]
        AgentCLI["代理 CLI<br/>(Claude, Codex, Cursor, ...)"]
        Desktop["Electron 桌面端"]
    end

    subgraph "外部服务"
        Vercel["Vercel<br/>(可选 web 托管)"]
        Anthropic["Anthropic API"]
        OpenAI["OpenAI API"]
        OtherAI["其他 AI API"]
    end

    Browser -->|"HTTP/SSE"| Web
    Web -->|"HTTP API"| Daemon
    Desktop -->|"Sidecar IPC"| Daemon
    Daemon -->|"读/写"| SQLite
    Daemon -->|"生成 & 流式传输"| AgentCLI
    AgentCLI -->|"stdout JSONL"| Daemon
    Daemon -->|"SSE 事件"| Web

    Vercel -.->|"可选"| Web
    Browser -->|"BYOK 代理"| Daemon
    Daemon -->|"SSRF 防护代理"| Anthropic
    Daemon -->|"SSRF 防护代理"| OpenAI
    Daemon -->|"SSRF 防护代理"| OtherAI
```

---

## 2. 部署拓扑

### 拓扑 A：完全本地（默认）

```mermaid
sequenceDiagram
    participant U as 用户
    participant B as 浏览器
    participant W as Next.js (localhost:3000)
    participant D as 守护进程 (localhost:7456)
    participant A as 代理 CLI (子进程)
    participant DB as SQLite

    U->>B: 打开应用
    B->>W: 加载 UI
    W->>D: GET /api/health
    D-->>W: 200 OK

    U->>B: 选择技能 + 编写提示词
    B->>D: POST /api/chat {skill, prompt, agent}
    D->>DB: 创建会话 + 消息
    D->>A: 生成代理进程 (stdin: 提示词)
    A-->>D: stdout JSONL 事件
    D-->>B: SSE 流 (content, tool_use, artifacts)
    B-->>U: 在沙箱 iframe 中渲染
```

### 拓扑 B：Vercel Web + 本地守护进程

```mermaid
sequenceDiagram
    participant U as 用户
    participant B as 浏览器
    participant V as Vercel Web
    participant D as 本地守护进程
    participant A as 代理 CLI

    U->>B: 打开应用 (vercel URL)
    B->>V: 加载 UI
    V->>D: WebSocket 隧道
    D-->>V: 状态 + API 响应
    U->>B: 聊天
    B->>D: POST /api/chat (通过隧道)
    D->>A: 生成代理
    A-->>D: stdout 事件
    D-->>B: SSE 流 (通过隧道)
```

### 拓扑 C：Vercel Web + 直连 API（无守护进程）

```mermaid
sequenceDiagram
    participant U as 用户
    participant B as 浏览器
    participant V as Vercel Serverless
    participant API as Anthropic/OpenAI API

    U->>B: 打开应用
    B->>V: 加载 UI
    U->>B: 聊天 (BYOK)
    B->>V: POST /api/proxy/anthropic/stream
    V->>API: 转发 (SSRF 防护)
    API-->>V: SSE 流
    V-->>B: 代理流
    B-->>U: 渲染制品
```

---

## 3. 数据流 — 从聊天到制品

```mermaid
flowchart LR
    A["用户编写提示词"] --> B["Web: POST /api/chat"]
    B --> C["守护进程: 解析代理 + 技能 + 设计系统"]
    C --> D["守护进程: 组合系统提示词"]
    D --> E["守护进程: 生成代理进程"]
    E --> F["代理: stdout JSONL 流"]
    F --> G["守护进程: 流解析器<br/>(claude-stream / json-event-stream)"]
    G --> H["守护进程: SSE 事件到 web"]
    H --> I["Web: 渲染内容块"]
    I --> J{"内容类型?"}
    J -->|"text/markdown"| K["在聊天中渲染"]
    J -->|"HTML 制品"| L["沙箱 iframe srcdoc"]
    J -->|"tool_use"| M["执行工具，回传结果"]
    M --> F
```

---

## 4. 模块依赖图

```mermaid
graph TD
    subgraph "应用"
        Web["@open-design/web"]
        Daemon["@open-design/daemon"]
        Desktop["@open-design/desktop"]
        Packaged["@open-design/packaged"]
    end

    subgraph "核心包"
        Contracts["@open-design/contracts"]
        Components["@open-design/components"]
        Sidecar["@open-design/sidecar"]
        SidecarProto["@open-design/sidecar-proto"]
        Platform["@open-design/platform"]
    end

    subgraph "协议包"
        Host["@open-design/host"]
        LauncherProto["@open-design/launcher-proto"]
        AguiAdapter["@open-design/agui-adapter"]
    end

    subgraph "工具包"
        PluginRuntime["@open-design/plugin-runtime"]
        RegistryProtocol["@open-design/registry-protocol"]
        Diagnostics["@open-design/diagnostics"]
        Download["@open-design/download"]
        Release["@open-design/release"]
        Metatool["@open-design/metatool"]
    end

    Web --> Contracts
    Web --> Components
    Daemon --> Contracts
    Daemon --> Platform
    Daemon --> PluginRuntime
    Desktop --> Sidecar
    Desktop --> SidecarProto
    Packaged --> Sidecar
    Packaged --> SidecarProto
    Sidecar --> SidecarProto
    Sidecar --> Platform
    Host --> Contracts
    AguiAdapter --> Contracts
    Diagnostics --> Platform
```

---

## 5. 守护进程内部架构

```mermaid
graph TD
    subgraph "守护进程 (Express Server)"
        HTTP["HTTP 服务器<br/>localhost:7456"]
        Routes["路由处理器<br/>/api/*"]
        DB["SQLite<br/>better-sqlite3"]
        RunTimes["代理运行时<br/>检测 + 启动 + 流式传输"]
        Plugins["插件系统<br/>清单 + 验证"]
        Skills["技能加载器<br/>SKILL.md 解析器"]
        DS["设计系统<br/>DESIGN.md 解析器"]
        Critique["评审引擎<br/>自评审"]
        Media["媒体适配器<br/>图片/视频/音频"]
        MCP["MCP 服务器<br/>stdio 协议"]
        Prompts["提示词组合器<br/>系统 + 技能 + DS"]
    end

    HTTP --> Routes
    Routes --> DB
    Routes --> RunTimes
    Routes --> Plugins
    Routes --> Skills
    Routes --> DS
    Routes --> Media
    RunTimes --> Prompts
    Prompts --> Skills
    Prompts --> DS
    RunTimes -->|"生成子进程"| AgentCLI["代理 CLI"]
    AgentCLI -->|"stdout"| RunTimes
    Critique -->|"评审制品"| DB
    MCP -->|"工具调用"| Routes
```

---

## 6. Sidecar IPC 架构

```mermaid
graph LR
    subgraph "打包应用"
        PackagedEntry["打包入口<br/>(esbuild bundle)"]
    end

    subgraph "Sidecar"
        SidecarRuntime["@open-design/sidecar<br/>IPC 服务器"]
        SidecarProto["@open-design/sidecar-proto<br/>协议模式"]
    end

    subgraph "守护进程"
        DaemonProcess["守护进程"]
        Stamp["进程戳<br/>app, mode, namespace,<br/>ipc, source"]
    end

    PackagedEntry -->|"Unix 套接字 /<br/>命名管道"| SidecarRuntime
    SidecarRuntime --> SidecarProto
    SidecarRuntime -->|"启动守护进程"| DaemonProcess
    DaemonProcess --> Stamp
    Stamp -->|"5 个字段"| SidecarRuntime
```

---

## 7. 插件生态系统架构

```mermaid
graph TD
    subgraph "插件规范"
        Spec["open-design.json<br/>清单"]
        SkillMd["SKILL.md<br/>技能定义"]
    end

    subgraph "插件运行时"
        Manifest["清单解析器<br/>(@open-design/plugin-runtime)"]
        Validator["模式验证器<br/>(Zod)"]
        Merger["配置合并器"]
        Resolver["引用解析器"]
    end

    subgraph "插件来源"
        Official["plugins/_official/<br/>261 个插件"]
        Community["plugins/community/"]
        Local["用户本地插件"]
    end

    subgraph "守护进程"
        PluginRoutes["插件路由<br/>/api/plugins"]
        PluginDB["插件表<br/>(SQLite)"]
    end

    Spec --> Manifest
    SkillMd --> Manifest
    Manifest --> Validator
    Validator --> Merger
    Merger --> Resolver
    Official --> PluginRoutes
    Community --> PluginRoutes
    Local --> PluginRoutes
    PluginRoutes --> PluginDB
    Resolver --> PluginDB
```

---

## 8. 技能与设计系统流程

```mermaid
sequenceDiagram
    participant U as 用户
    participant W as Web
    participant D as 守护进程
    participant S as 技能 (SKILL.md)
    participant DS as 设计系统 (DESIGN.md)
    participant A as 代理 CLI

    U->>W: 选择技能 "deck-creator"
    W->>D: GET /api/skills
    D->>S: 读取 SKILL.md + od: frontmatter
    D-->>W: 技能列表及元数据

    U->>W: 选择设计系统 "Linear"
    W->>D: GET /api/design-systems
    D->>DS: 读取 DESIGN.md（9 个部分）
    D-->>W: 设计系统列表

    U->>W: 提交提示词
    W->>D: POST /api/chat
    D->>S: 加载技能指令
    D->>DS: 加载设计约束
    D->>D: 组合系统提示词<br/>(技能 + DS + 工艺规则 + 上下文)
    D->>A: 使用组合提示词生成代理
    A-->>D: 生成 HTML 制品
    D-->>W: SSE: 制品内容
    W-->>U: 在沙箱 iframe 中渲染
```

---

## 9. 代理流处理

```mermaid
flowchart TD
    A["代理 CLI stdout"] --> B{"代理类型?"}
    B -->|"Claude"| C["claude-stream.ts<br/>JSONL 解析器"]
    B -->|"通用"| D["json-event-stream.ts<br/>NDJSON 解析器"]
    B -->|"Qoder"| E["qoder-stream.ts<br/>自定义解析器"]

    C --> F["事件标准化"]
    D --> F
    E --> F

    F --> G{"事件类型?"}
    G -->|"content_block_start"| H["开始内容块"]
    G -->|"content_block_delta"| I["追加内容"]
    G -->|"content_block_stop"| J["完成内容块"]
    G -->|"tool_use"| K["注册工具调用"]
    G -->|"tool_result"| L["回传结果给代理"]
    G -->|"turn_end"| M["如果没有待处理答案则关闭 stdin"]

    H --> N["SSE 到 web"]
    I --> N
    J --> N
    K --> N
    L -->|"写入代理 stdin"| A
    M --> N
```

---

## 10. 关键架构决策

### 10.1 代理原生、模型无关

Open Design **不**附带自己的 AI 模型或代理。它发现用户 PATH 中的编码代理 CLI 并编排它们。这意味着：
- 用户自带 API 密钥和模型访问权限
- 系统适用于任何输出 JSONL/NDJSON stdout 的代理
- 代理功能在运行时通过 `capabilities.ts` 发现

### 10.2 技能驱动设计

每个设计任务由一个 **SKILL.md** 文件驱动 — 一个带有 YAML frontmatter 的 Markdown 文档，定义：
- 技能的模式（prototype、deck、image 等）
- 所需的工艺规则
- 提示词模板
- 输出格式期望

代理在系统提示词中读取 SKILL.md 并遵循其指令。

### 10.3 设计系统契约

品牌一致性通过 **DESIGN.md** 文件强制执行，包含 9 个部分的模式：
1. 调色板
2. 排版比例
3. 间距系统
4. 布局网格
5. 组件库
6. 动效/动画
7. 语调
8. 品牌指南
9. 反模式

### 10.4 沙箱制品预览

所有生成的制品（HTML、SVG 等）在沙箱 iframe 中渲染：
```html
<iframe sandbox="allow-scripts" srcdoc="..."></iframe>
```
这防止不受信任的生成代码访问宿主页面、localStorage 或发起任意网络请求。

### 10.5 SQLite 作为唯一数据源

守护进程使用单个 SQLite 文件（`app.sqlite`），采用 WAL 模式存储所有持久状态：项目、会话、消息、插件、媒体任务。这使系统完全本地化，无需外部数据库依赖。

### 10.6 能力双轨制

每个面向用户的功能必须通过 **Web UI** 和 **`od` CLI** 两种方式可访问。两种表面调用相同的 `/api/*` 端点。CLI 支持 `--json` 用于机器可读输出，支持 `--prompt-file` 用于管道输入。

### 10.7 MCP 服务器集成

守护进程通过 stdio 暴露 MCP（模型上下文协议）服务器，可安装到任何兼容 MCP 的代理中。这允许外部代理通过 MCP 工具调用调用 Open Design 的工具（制品渲染、设计系统查询等）。

---

## 11. 发布频道模型

```mermaid
graph LR
    Beta["Beta 频道<br/>(日常研发)"]
    Nightly["Nightly 频道<br/>(内部验证)"]
    Preview["Preview 频道<br/>(抢先体验)"]
    Stable["Stable 频道<br/>(正式发布)"]

    Beta -->|"验证"| Nightly
    Nightly -->|"门控"| Stable
    Preview -.->|"独立"| Stable
```

| 频道 | 应用标识 | 用途 |
|---|---|---|
| `beta` | Open Design Beta | 日常研发/开发验证 |
| `nightly` | Open Design | 稳定发布的内部验证 |
| `preview` | Open Design Preview | 具有稳定版严谨度的抢先体验 |
| `stable` | Open Design | 正式发布 |

---

## 12. 国际化架构

- 支持 **24 种语言**：ar、de、en、es-ES、fa、fr、hu、id、ja、ko、pl、pt-BR、ru、th、tr、uk、zh-CN、zh-TW 等
- `apps/web/src/i18n/types.ts` 中的类型化 `Dict`
- 每种语言是 `apps/web/src/i18n/locales/` 下的独立 `.ts` 文件
- 缺失的键会产生类型检查错误
