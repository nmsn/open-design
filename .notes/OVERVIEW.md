# Open Design — Full Codebase Overview

> Auto-generated on 2026-06-25. Covers v0.11.1.

---

## 1. Project Identity

| | |
|---|---|
| **Name** | `open-design` |
| **Version** | 0.11.1 |
| **License** | Apache-2.0 |
| **Description** | Local-first, open-source Claude Design alternative — a design workspace that detects installed coding-agent CLIs, runs design skills + design systems, and streams artifacts into a sandboxed preview. |
| **Package Manager** | pnpm 10.33.2 (via Corepack) |
| **Runtime Target** | Node ~24 |
| **Module System** | ESM (`"type": "module"` everywhere) |

---

## 2. Tech Stack

### Languages

| Language | Usage |
|---|---|
| **TypeScript** | Primary (~100% of source code) |
| **JavaScript** | Generated/vendored output, bin shims only |
| **CSS** | Tailwind CSS 4.3.0 via PostCSS + CSS Modules |

### Frameworks & Libraries

| Layer | Stack |
|---|---|
| Frontend | **Next.js 16** (App Router) + **React 18** |
| Styling | **Tailwind CSS 4.3** + CSS Modules |
| Animation | **Motion** (Framer Motion successor) 12.40 |
| Rich Text | **Lexical** 0.36.2 |
| Syntax Highlighting | **Shiki** 4.1 |
| Terminal | **xterm.js** 5.5 |
| Backend | **Express 5.2** (daemon HTTP server) |
| Database | **better-sqlite3** 12.10 (SQLite, WAL mode) |
| Streaming | **Server-Sent Events** (SSE) for agent output |
| Desktop | **Electron** 41.3 |
| Build | **esbuild** 0.28 (packages), **Next.js** (web), **tsc** (type checking), **electron-builder** 26.8 (packaging) |
| Testing | **Vitest** 4.1.6 (unit/integration), **Playwright** 1.60 (E2E/UI), **pixelmatch** (visual regression) |
| MCP | **@modelcontextprotocol/sdk** 1.29 |
| AI SDKs | **@anthropic-ai/sdk** 0.32, **openai** 6.38 |
| Telemetry | **PostHog** (browser + node), **OpenTelemetry** |
| Monitoring | **prom-client** (Prometheus metrics) |
| Observability | **Langfuse** (tracing bridge) |

### Key Dependencies

| Dependency | Purpose |
|---|---|
| `zod` 3.25 | Schema validation |
| `jszip` | ZIP export |
| `multer` 2.1 | File uploads |
| `cheerio` 1.2 | HTML parsing |
| `chokidar` 5.0 | File watching |
| `node-pty` 1.1 | Terminal emulation |
| `tar` 7.5 | Archive handling |
| `blake3-wasm` 2.1 | Content hashing |
| `@powerformer/vela-cli` | Optional AMR CLI |

---

## 3. Root-Level Configuration Files

| File | Purpose |
|---|---|
| `package.json` | Root workspace package, defines `od` bin entry at `apps/daemon/bin/od.mjs`, scripts (`guard`, `typecheck`, `tools-dev`, `tools-pack`, etc.) |
| `pnpm-workspace.yaml` | Workspace packages: `packages/*`, `apps/*`, `tools/*`, `e2e` |
| `.node-version` | Contains `24` |
| `vercel.json` | Vercel deployment config: builds `@open-design/web`, outputs to `apps/web/out` |
| `flake.nix` | Nix flake for reproducible dev environments |
| `mise.toml` | Mise tool version manager config |
| `.gitignore` | Standard ignores including `.tmp/`, `.next/`, `dist/` |
| `.dockerignore` | Docker build exclusions |
| `AGENTS.md` | Master directory guide for AI agents — the single source of truth for repository conventions |
| `CLAUDE.md` | Minimal Claude Code config |
| `CONTEXT.md` | Domain glossary defining Project, Artifact, Live Artifact, etc. |

> **Note:** There is no `tsconfig.json` at the root. Each package/app has its own.

---

## 4. Directory Structure

### 4.1 `apps/` — Application Packages (6 apps)

| Directory | Package Name | Role |
|---|---|---|
| `apps/web` | `@open-design/web` | Next.js 16 App Router + React 18 frontend |
| `apps/daemon` | `@open-design/daemon` | Local privileged daemon (Express + SQLite), the `od` CLI |
| `apps/desktop` | `@open-design/desktop` | Electron shell (Electron 41.3) |
| `apps/packaged` | `@open-design/packaged` | Packaged Electron runtime entry, esbuild-bundled |
| `apps/landing-page` | — | Marketing landing page |
| `apps/telemetry-worker` | — | Telemetry processing worker |

### 4.2 `packages/` — Shared Libraries (15 packages)

| Package | Purpose |
|---|---|
| `@open-design/contracts` | Pure TypeScript API contracts (DTOs, SSE events, error shapes) shared between web and daemon. Uses Zod for validation. |
| `@open-design/components` | Shared React UI primitives (Button, Dialog, Input, Select, Textarea, VisuallyHidden) |
| `@open-design/sidecar` | Generic sidecar runtime — IPC server/client (Unix sockets / Windows named pipes), port allocation, namespace resolution, runtime path management |
| `@open-design/sidecar-proto` | Open Design sidecar business protocol definitions |
| `@open-design/platform` | OS process primitives — spawn, kill, proxy detection, process stamp matching, user toolchain bin discovery |
| `@open-design/host` | Renderer host bridge protocol |
| `@open-design/launcher-proto` | Launcher protocol definitions |
| `@open-design/release` | Release domain primitives |
| `@open-design/diagnostics` | Log collection, redaction, zip packaging for diagnostics export |
| `@open-design/download` | Download handling |
| `@open-design/agui-adapter` | Bidirectional adapter between OD events and AG-UI protocol (CopilotKit) |
| `@open-design/plugin-runtime` | Plugin manifest parsing, validation, merging, ref resolution (pure TS, no node:fs) |
| `@open-design/registry-protocol` | Plugin registry backend protocol |
| `@open-design/metatool` | Internal metadata helpers for build freshness |

### 4.3 `tools/` — Build/Lifecycle Tools (4 tools)

| Tool | Package Name | Purpose |
|---|---|---|
| `tools/dev` | `@open-design/tools-dev` | Local dev lifecycle: `start`/`stop`/`run`/`status`/`logs`/`inspect`/`check`. Entry via `pnpm tools-dev`. |
| `tools/pack` | `@open-design/tools-pack` | Packaged Electron builds: electron-builder, notarization, installer identity validation, beta release preparation |
| `tools/serve` | `@open-design/tools-serve` | Fixture service: deterministic updater metadata and artifacts |
| `tools/release` | `@open-design/tools-release` | Release automation |

### 4.4 `e2e/` — End-to-End Tests

- **Playwright** for UI automation, **Vitest** for API/spec tests
- Priority levels: P0, P1, P2
- Test directories: `tests/`, `specs/`, `ui/`
- Visual regression testing support (`playwright.visual.config.ts`)
- Resources in `resources/`, shared lib in `lib/`

### 4.5 `skills/` — Agent Design Skills (~160 folders)

- Each skill is a folder with `SKILL.md` + supporting files
- Follows Claude Code SKILL.md convention with `od:` frontmatter extensions
- **Modes:** `prototype`, `deck`, `image`, `video`, `audio`, `template`, `design-system`, `utility`
- **Scenarios:** `design`, `marketing`, `operation`, `engineering`, `product`, `finance`, `hr`, `sale`, `personal`

### 4.6 `design-templates/` — Rendering Catalogue (~113 templates)

- Deck templates: `html-ppt-*` (15 templates × 36 themes), `guizang-ppt`
- Prototype templates: `web-prototype`, `mobile-app`, `dashboard`, `saas-landing`, etc.
- Video: `hyperframes/` (HTML to MP4 motion graphics)
- Image/video/audio templates

### 4.7 `design-systems/` — Brand DESIGN.md Systems (~152 brands)

- Each is a folder with a `DESIGN.md` file
- **9-section schema:** color, typography, spacing, layout, components, motion, voice, brand, anti-patterns
- Covers AI, developer tools, fintech, e-commerce, media, automotive, etc.

### 4.8 `craft/` — Universal Brand-Agnostic Craft Rules

- Markdown files covering: accessibility baseline, animation discipline, anti-AI slop, color, form validation, laws of UX, RTL/bidi, state coverage, typography hierarchy
- Skills opt in via `od.craft.requires`

### 4.9 `mocks/` — Replay-Based Mock CLIs

- Mock agents for: opencode, claude, codex, gemini, cursor-agent, deepseek, qwen, grok, devin, hermes, kilo, kimi, kiro, vibe, vela (AMR)
- Built from anonymized Langfuse traces
- Used for tests and self-validation
- Contains `manifest.json`, `mock-agent.mjs`, `golden/` fixtures

### 4.10 `plugins/` — Plugin Ecosystem

- `_official/`: 261 official plugins (scenarios, image-templates, video-templates, design-systems, atoms, examples)
- `community/`: Community plugins
- `registry/`: Publishing flow
- `spec/`: Plugin specification (SPEC.md, AGENT-DEVELOPMENT.md, CONTRIBUTING.md, PUBLISHING-REGISTRIES.md)

### 4.11 Other Directories

| Directory | Purpose |
|---|---|
| `docs/` | Architecture, protocols, agent adapters, modes, i18n, deployment guides |
| `deploy/` | Docker/Docker Compose, AWS, Azure deployment configs |
| `scripts/` | Build/guard/seed/sync scripts |
| `prompt-templates/` | Image/video prompt templates |
| `data/`, `specs/`, `templates/`, `story/`, `assets/`, `nix/`, `clipper/`, `figma-plugin/` | Various supporting directories |

---

## 5. Database Schema (SQLite)

The daemon uses a single SQLite database (`app.sqlite`) with WAL mode. Key tables:

| Table | Columns | Purpose |
|---|---|---|
| `projects` | id, name, skill_id, design_system_id, pending_prompt, metadata_json, timestamps | Project registry |
| `templates` | id, name, description, source_project_id, files_json | Reusable templates |
| `conversations` | id, project_id, title, session_mode, timestamps | Chat sessions (FK → projects) |
| `agent_sessions` | conversation_id, agent_id, session_id, stable_prompt_hash | Agent session tracking |
| `messages` | id, conversation_id, role, content, agent_id, events_json, attachments_json, produced_files_json, feedback_json, run_context_json, applied_plugin_snapshot_json, timestamps | Chat messages |
| `preview_comments` | id, project_id, conversation_id, file_path, element_id, selector, label, note, status, slide_index, timestamps | Artifact review comments |
| `tabs` | project_id | Workspace tabs |

Additional tables exist for critique, media tasks, library, and plugins (managed via migration functions in `apps/daemon/src/db.ts`).

---

## 6. API Routes

The daemon exposes HTTP routes under `/api/*` on `localhost:7456`. Key routes:

| Route | Method | Purpose |
|---|---|---|
| `/api/health` | GET | Health check |
| `/api/agents` | GET | List detected coding-agent CLIs |
| `/api/skills` | GET | List available skills |
| `/api/design-systems` | GET | List design systems |
| `/api/projects` | GET/POST | CRUD projects |
| `/api/import/folder` | POST | Import existing folder as project |
| `/api/import/claude-design` | POST | Import Claude Design ZIP |
| `/api/projects/:id/files` | GET | Project file listing |
| `/api/projects/:id/upload` | POST | File upload |
| `/api/chat` | POST | SSE streaming chat (agent output) |
| `/api/proxy/{provider}/stream` | POST | BYOK proxy (SSRF-guarded) for anthropic, openai, azure, google, ollama, senseaudio |
| `/api/artifacts/save` | POST | Save artifact |
| `/api/artifacts/lint` | POST | Lint artifact |
| `/api/plugins` | GET | Plugin marketplace |
| `/api/routines` | GET | Automation routines |
| `/api/version` | GET | Version info |

**Route files** in `apps/daemon/src/routes/`:
`active-context.ts`, `automation.ts`, `chat.ts`, `daemon.ts`, `deploy.ts`, `design-system-tool.ts`, `design-systems.ts`, `genui.ts`, `handoff.ts`, `host-tools.ts`, `library.ts`, `live-artifact.ts`, `media.ts`, `memory.ts`, `plugins/`, `project/`, `routine.ts`, `runs.ts`, `social-share.ts`, `static-resource.ts`, `telemetry.ts`, `terminal.ts`, `vela.ts`, `xai.ts`

---

## 7. Agent Adapter System

The daemon supports **22+ coding-agent CLIs** via adapters in `apps/daemon/src/runtimes/`:

| Module | Purpose |
|---|---|
| `detection.ts`, `resolution.ts` | PATH scan + config-dir probe to discover installed agents |
| `registry.ts` | Agent definitions, profile management |
| `launch.ts`, `env.ts` | Spawn agent processes with standardized env |
| `claude-stream.ts`, `json-event-stream.ts`, `qoder-stream.ts` | Parse agent stdout into structured events |
| `capabilities.ts` | Feature matrix per agent |
| `mcp.ts` | Live artifact MCP server configuration |
| `models.ts` | Model registry, AMR integration |

### Supported Agents

Claude Code, Codex, Cursor, Copilot, OpenClaw, Antigravity, Gemini CLI, OpenCode, Qwen, Qoder, Hermes, Kimi, Pi Agent, Mistral Vibe, Cline, Trae, DeepSeek, Kiro, Kilo, Devin

---

## 8. Entry Points

| Entry | File | Purpose |
|---|---|---|
| `od` CLI | `apps/daemon/bin/od.mjs` | Main CLI binary |
| Daemon server | `apps/daemon/src/server.ts` | Express server, all `/api/*` routes |
| CLI commands | `apps/daemon/src/cli.ts` | All `od` subcommands |
| Web app | `apps/web/app/layout.tsx` + `app/[[...slug]]/` | Next.js App Router entry |
| Main React app | `apps/web/src/App.tsx` | Primary React application |
| Desktop main | `apps/desktop/src/main/index.ts` | Electron main process |
| Packaged entry | `apps/packaged/dist/index.mjs` | esbuild-bundled runtime |
| tools-dev | `tools/dev/bin/tools-dev.mjs` | Dev lifecycle CLI |
| tools-pack | `tools/pack/bin/tools-pack.mjs` | Build/package CLI |

---

## 9. Key Source Directories — Detail

### Daemon (`apps/daemon/src/`) — ~140 files

| Subdirectory | Purpose |
|---|---|
| `server.ts`, `cli.ts` | Main server and CLI (each ~9K lines) |
| `db.ts` | SQLite schema and queries |
| `runtimes/` | 30+ files: agent detection, spawning, streaming |
| `routes/` | 25+ route modules |
| `plugins/` | 40+ plugin management files |
| `critique/` | 18 files: artifact self-critique system |
| `artifacts/` | Artifact handling |
| `design-systems/` | Design system management |
| `integrations/` | External integrations |
| `media/`, `media-adapters/` | Media generation |
| `prompts/` | System prompt composition |
| `memory.ts` | Project memory/context |

### Web (`apps/web/src/`) — ~27 subdirectories

| Subdirectory | Purpose |
|---|---|
| `App.tsx` | Main app component |
| `components/` | 180+ React components |
| `runtime/` | Agent runtime integration |
| `state/` | Application state management |
| `providers/` | React context providers |
| `i18n/` | Internationalization (24 locale files) |
| `artifacts/` | Artifact display |
| `analytics/` | Analytics integration |
| `edit-mode/` | In-place editing |
| `hooks/`, `lib/`, `utils/` | Shared utilities |

---

## 10. Build & Development Workflow

| Command | Purpose |
|---|---|
| `pnpm tools-dev run web` | Start dev environment (Next.js + daemon) |
| `pnpm guard` | Style policy, product neutrality, import isolation checks |
| `pnpm typecheck` | Type check all packages |
| `pnpm --filter @open-design/web build` | Build web app |
| `pnpm --filter @open-design/daemon build` | Build daemon |
| `pnpm --filter @open-design/desktop build` | Build desktop |
| `pnpm --filter @open-design/web test` | Run web tests |
| `pnpm --filter @open-design/daemon test` | Run daemon tests |
| `pnpm tools-pack mac build --to all` | Build macOS package |
| `pnpm tools-pack win build --to nsis` | Build Windows package |
| `pnpm tools-pack linux build --to appimage` | Build Linux package |
