# Open Design — Codebase Documentation

> Auto-generated on 2026-06-25. Covers v0.11.1.

## What is Open Design?

Open Design is a **local-first, open-source design workspace** — a Claude Design alternative that detects installed coding-agent CLIs, runs design skills + design systems, and streams artifacts into a sandboxed preview.

It is **agent-native and model-agnostic**: it does not ship its own AI agent. Instead, it discovers agent CLIs already on the user's PATH (Claude Code, Codex, Cursor, Gemini CLI, etc.) and orchestrates them through a unified skill-driven protocol.

## Documentation Index

| Document | Description |
|---|---|
| [OVERVIEW.md](./OVERVIEW.md) | Full project overview — tech stack, directory structure, database schema, API routes, agent adapters, entry points |
| [ARCHITECTURE.md](./ARCHITECTURE.md) | Architecture diagrams, data flows, deployment topologies, design decisions |

## Quick Links

- **Root AGENTS.md**: `../../AGENTS.md` — the single source of truth for repository conventions
- **Architecture docs**: `../architecture.md`, `../spec.md`, `../modes.md`
- **Plugin spec**: `../../plugins/spec/SPEC.md`
- **Contributing**: `../../CONTRIBUTING.md`

## At a Glance

| | |
|---|---|
| **Version** | 0.11.1 |
| **License** | Apache-2.0 |
| **Language** | TypeScript (ESM) |
| **Runtime** | Node ~24 |
| **Package Manager** | pnpm 10.33.2 (Corepack) |
| **Frontend** | Next.js 16 + React 18 + Tailwind CSS 4.3 |
| **Backend** | Express 5.2 + SQLite (better-sqlite3) |
| **Desktop** | Electron 41.3 |
| **Testing** | Vitest 4.1.6 + Playwright 1.60 |
| **CI/CD** | GitHub Actions, Vercel (web), electron-builder (desktop) |
