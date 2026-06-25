# Open Design — Architecture

> Auto-generated on 2026-06-25. Covers v0.11.1.

---

## 1. High-Level Architecture

```mermaid
graph TD
    subgraph "User's Machine"
        Browser["Browser"]
        Web["Next.js Web App<br/>localhost:3000"]
        Daemon["Daemon (Express)<br/>localhost:7456"]
        SQLite[("SQLite<br/>app.sqlite")]
        AgentCLI["Agent CLIs<br/>(Claude, Codex, Cursor, ...)"]
        Desktop["Electron Desktop"]
    end

    subgraph "External Services"
        Vercel["Vercel<br/>(optional web hosting)"]
        Anthropic["Anthropic API"]
        OpenAI["OpenAI API"]
        OtherAI["Other AI APIs"]
    end

    Browser -->|"HTTP/SSE"| Web
    Web -->|"HTTP API"| Daemon
    Desktop -->|"Sidecar IPC"| Daemon
    Daemon -->|"Read/Write"| SQLite
    Daemon -->|"Spawn & Stream"| AgentCLI
    AgentCLI -->|"stdout JSONL"| Daemon
    Daemon -->|"SSE Events"| Web

    Vercel -.->|"Optional"| Web
    Browser -->|"BYOK Proxy"| Daemon
    Daemon -->|"SSRF-guarded proxy"| Anthropic
    Daemon -->|"SSRF-guarded proxy"| OpenAI
    Daemon -->|"SSRF-guarded proxy"| OtherAI
```

---

## 2. Deployment Topologies

### Topology A: Fully Local (Default)

```mermaid
sequenceDiagram
    participant U as User
    participant B as Browser
    participant W as Next.js (localhost:3000)
    participant D as Daemon (localhost:7456)
    participant A as Agent CLI (child process)
    participant DB as SQLite

    U->>B: Opens app
    B->>W: Load UI
    W->>D: GET /api/health
    D-->>W: 200 OK

    U->>B: Selects skill + writes prompt
    B->>D: POST /api/chat {skill, prompt, agent}
    D->>DB: Create conversation + message
    D->>A: Spawn agent process (stdin: prompt)
    A-->>D: stdout JSONL events
    D-->>B: SSE stream (content, tool_use, artifacts)
    B-->>U: Render in sandboxed iframe
```

### Topology B: Web on Vercel + Local Daemon

```mermaid
sequenceDiagram
    participant U as User
    participant B as Browser
    participant V as Vercel Web
    participant D as Local Daemon
    participant A as Agent CLI

    U->>B: Opens app (vercel URL)
    B->>V: Load UI
    V->>D: WebSocket tunnel
    D-->>V: Status + API responses
    U->>B: Chat
    B->>D: POST /api/chat (via tunnel)
    D->>A: Spawn agent
    A-->>D: stdout events
    D-->>B: SSE stream (via tunnel)
```

### Topology C: Web on Vercel + Direct API (No Daemon)

```mermaid
sequenceDiagram
    participant U as User
    participant B as Browser
    participant V as Vercel Serverless
    participant API as Anthropic/OpenAI API

    U->>B: Opens app
    B->>V: Load UI
    U->>B: Chat (BYOK)
    B->>V: POST /api/proxy/anthropic/stream
    V->>API: Forward (SSRF-guarded)
    API-->>V: SSE stream
    V-->>B: Proxy stream
    B-->>U: Render artifacts
```

---

## 3. Data Flow — Chat to Artifact

```mermaid
flowchart LR
    A["User writes prompt"] --> B["Web: POST /api/chat"]
    B --> C["Daemon: resolve agent + skill + design system"]
    C --> D["Daemon: compose system prompt"]
    D --> E["Daemon: spawn agent process"]
    E --> F["Agent: stdout JSONL stream"]
    F --> G["Daemon: stream parser<br/>(claude-stream / json-event-stream)"]
    G --> H["Daemon: SSE events to web"]
    H --> I["Web: render content blocks"]
    I --> J{"Content type?"}
    J -->|"text/markdown"| K["Render in chat"]
    J -->|"HTML artifact"| L["Sandboxed iframe srcdoc"]
    J -->|"tool_use"| M["Execute tool, feed result back"]
    M --> F
```

---

## 4. Module Dependency Graph

```mermaid
graph TD
    subgraph "Apps"
        Web["@open-design/web"]
        Daemon["@open-design/daemon"]
        Desktop["@open-design/desktop"]
        Packaged["@open-design/packaged"]
    end

    subgraph "Core Packages"
        Contracts["@open-design/contracts"]
        Components["@open-design/components"]
        Sidecar["@open-design/sidecar"]
        SidecarProto["@open-design/sidecar-proto"]
        Platform["@open-design/platform"]
    end

    subgraph "Protocol Packages"
        Host["@open-design/host"]
        LauncherProto["@open-design/launcher-proto"]
        AguiAdapter["@open-design/agui-adapter"]
    end

    subgraph "Utility Packages"
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

## 5. Daemon Internal Architecture

```mermaid
graph TD
    subgraph "Daemon (Express Server)"
        HTTP["HTTP Server<br/>localhost:7456"]
        Routes["Route Handlers<br/>/api/*"]
        DB["SQLite<br/>better-sqlite3"]
        RunTimes["Agent Runtimes<br/>Detection + Launch + Stream"]
        Plugins["Plugin System<br/>Manifest + Validation"]
        Skills["Skill Loader<br/>SKILL.md parser"]
        DS["Design Systems<br/>DESIGN.md parser"]
        Critique["Critique Engine<br/>Self-review"]
        Media["Media Adapters<br/>Image/Video/Audio"]
        MCP["MCP Server<br/>stdio protocol"]
        Prompts["Prompt Composer<br/>System + Skill + DS"]
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
    RunTimes -->|"Spawn child"| AgentCLI["Agent CLIs"]
    AgentCLI -->|"stdout"| RunTimes
    Critique -->|"Review artifacts"| DB
    MCP -->|"Tool calls"| Routes
```

---

## 6. Sidecar IPC Architecture

```mermaid
graph LR
    subgraph "Packaged App"
        PackagedEntry["Packaged Entry<br/>(esbuild bundle)"]
    end

    subgraph "Sidecar"
        SidecarRuntime["@open-design/sidecar<br/>IPC Server"]
        SidecarProto["@open-design/sidecar-proto<br/>Protocol Schema"]
    end

    subgraph "Daemon"
        DaemonProcess["Daemon Process"]
        Stamp["Process Stamp<br/>app, mode, namespace,<br/>ipc, source"]
    end

    PackagedEntry -->|"Unix socket /<br/>Named pipe"| SidecarRuntime
    SidecarRuntime --> SidecarProto
    SidecarRuntime -->|"Launch daemon"| DaemonProcess
    DaemonProcess --> Stamp
    Stamp -->|"5 fields"| SidecarRuntime
```

---

## 7. Plugin Ecosystem Architecture

```mermaid
graph TD
    subgraph "Plugin Spec"
        Spec["open-design.json<br/>Manifest"]
        SkillMd["SKILL.md<br/>Skill Definition"]
    end

    subgraph "Plugin Runtime"
        Manifest["Manifest Parser<br/>(@open-design/plugin-runtime)"]
        Validator["Schema Validator<br/>(Zod)"]
        Merger["Config Merger"]
        Resolver["Ref Resolver"]
    end

    subgraph "Plugin Sources"
        Official["plugins/_official/<br/>261 plugins"]
        Community["plugins/community/"]
        Local["User local plugins"]
    end

    subgraph "Daemon"
        PluginRoutes["Plugin Routes<br/>/api/plugins"]
        PluginDB["Plugin Tables<br/>(SQLite)"]
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

## 8. Skill & Design System Flow

```mermaid
sequenceDiagram
    participant U as User
    participant W as Web
    participant D as Daemon
    participant S as Skill (SKILL.md)
    participant DS as Design System (DESIGN.md)
    participant A as Agent CLI

    U->>W: Select skill "deck-creator"
    W->>D: GET /api/skills
    D->>S: Read SKILL.md + od: frontmatter
    D-->>W: Skill list with metadata

    U->>W: Select design system "Linear"
    W->>D: GET /api/design-systems
    D->>DS: Read DESIGN.md (9 sections)
    D-->>W: Design system list

    U->>W: Submit prompt
    W->>D: POST /api/chat
    D->>S: Load skill instructions
    D->>DS: Load design constraints
    D->>D: Compose system prompt<br/>(skill + DS + craft rules + context)
    D->>A: Spawn with composed prompt
    A-->>D: Generate HTML artifact
    D-->>W: SSE: artifact content
    W-->>U: Render in sandboxed iframe
```

---

## 9. Agent Stream Processing

```mermaid
flowchart TD
    A["Agent CLI stdout"] --> B{"Agent type?"}
    B -->|"Claude"| C["claude-stream.ts<br/>JSONL parser"]
    B -->|"Generic"| D["json-event-stream.ts<br/>NDJSON parser"]
    B -->|"Qoder"| E["qoder-stream.ts<br/>Custom parser"]

    C --> F["Event normalization"]
    D --> F
    E --> F

    F --> G{"Event type?"}
    G -->|"content_block_start"| H["Start content chunk"]
    G -->|"content_block_delta"| I["Append to content"]
    G -->|"content_block_stop"| J["Finalize content block"]
    G -->|"tool_use"| K["Register tool call"]
    G -->|"tool_result"| L["Feed result back to agent"]
    G -->|"turn_end"| M["Close stdin if no pending answers"]

    H --> N["SSE to web"]
    I --> N
    J --> N
    K --> N
    L -->|"Write to agent stdin"| A
    M --> N
```

---

## 10. Key Architectural Decisions

### 10.1 Agent-Native, Model-Agnostic

Open Design does **not** ship its own AI model or agent. It discovers coding-agent CLIs on the user's PATH and orchestrates them. This means:
- Users bring their own API keys and model access
- The system works with any agent that speaks JSONL/NDJSON stdout
- Agent capabilities are discovered at runtime via `capabilities.ts`

### 10.2 Skill-Driven Design

Every design task is driven by a **SKILL.md** file — a Markdown document with YAML frontmatter that defines:
- The skill's mode (prototype, deck, image, etc.)
- Required craft rules
- Prompt templates
- Output format expectations

Agents read SKILL.md as part of their system prompt and follow its instructions.

### 10.3 Design System Contracts

Brand consistency is enforced through **DESIGN.md** files with a 9-section schema:
1. Color palette
2. Typography scale
3. Spacing system
4. Layout grid
5. Component library
6. Motion/animation
7. Voice/tone
8. Brand guidelines
9. Anti-patterns

### 10.4 Sandboxed Artifact Preview

All generated artifacts (HTML, SVG, etc.) are rendered in a sandboxed iframe:
```html
<iframe sandbox="allow-scripts" srcdoc="..."></iframe>
```
This prevents untrusted generated code from accessing the host page, localStorage, or making arbitrary network requests.

### 10.5 SQLite as Single Source of Truth

The daemon uses a single SQLite file (`app.sqlite`) in WAL mode for all persistent state: projects, conversations, messages, plugins, media tasks. This keeps the system fully local with no external database dependency.

### 10.6 Capability Dual-Track

Every user-facing feature must be accessible through **both** the web UI **and** the `od` CLI. Both surfaces call the same `/api/*` endpoints. The CLI supports `--json` for machine-readable output and `--prompt-file` for piped input.

### 10.7 MCP Server Integration

The daemon exposes an MCP (Model Context Protocol) server over stdio, installable into any MCP-compatible agent. This allows external agents to call Open Design's tools (artifact rendering, design system lookup, etc.) as MCP tool calls.

---

## 11. Release Channel Model

```mermaid
graph LR
    Beta["Beta Channel<br/>(daily R&D)"]
    Nightly["Nightly Channel<br/>(internal validation)"]
    Preview["Preview Channel<br/>(early access)"]
    Stable["Stable Channel<br/>(formal delivery)"]

    Beta -->|"Validate"| Nightly
    Nightly -->|"Gate"| Stable
    Preview -.->|"Independent"| Stable
```

| Channel | App Identity | Purpose |
|---|---|---|
| `beta` | Open Design Beta | Daily R&D/development validation |
| `nightly` | Open Design | Internal validation for stable delivery |
| `preview` | Open Design Preview | Early-access with stable-like rigor |
| `stable` | Open Design | Formal delivery |

---

## 12. i18n Architecture

- **24 locales** supported: ar, de, en, es-ES, fa, fr, hu, id, ja, ko, pl, pt-BR, ru, th, tr, uk, zh-CN, zh-TW, and more
- Typed `Dict` in `apps/web/src/i18n/types.ts`
- Each locale is a separate `.ts` file under `apps/web/src/i18n/locales/`
- Missing keys produce typecheck errors
