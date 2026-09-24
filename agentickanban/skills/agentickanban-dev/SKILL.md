---
name: agentickanban-dev
description: Develop on the agentic-kanban codebase — setup, run, architecture map, mandatory conventions, quality gates and where the project knowledge lives. Use when the user wants to build, run, test, debug or modify agentic-kanban locally.
---

# Develop on agentic-kanban

Practical, opinionated guide for working on the `agentic-kanban` codebase
(p-wegner/agentic-kanban, TypeScript pnpm monorepo, MIT). Full guide:
[references/development-guide.md](references/development-guide.md). Conventions digest:
[references/conventions.md](references/conventions.md).

## Setup (Node LTS 22 — never 23; pnpm 10)

```bash
pnpm install
pnpm db:setup        # migrate + seed + register this repo as a project
pnpm dev             # server :3001 + client :5173 → open http://localhost:5173
```

Clean-clone gotchas live in repo `docs/install.md`. Do NOT run setup scripts without asking
the user first (they mutate state; `pnpm db:*` touches `kanban.db` — never delete it).

## Monorepo map

| Package | Role |
|---|---|
| `packages/server` | REST API + WS board events, agent execution, workspaces (git worktrees), merge queue/train, risk posture, worker fleet |
| `packages/client` | React + Vite board UI (optimistic mutations, live reconciliation) |
| `packages/shared` | Types/schema + single-source-of-truth services (git-service, profile roster) |
| `packages/mcp-server` | MCP server (~35 tools) exposing the board to coding agents |
| `packages/desktop` | Tauri v2 wrapper |
| `packages/e2e` | Playwright suites |

## Load-bearing rules (violating these breaks guarded invariants)

- All high-level git goes through one `git-service.ts`; the only sanctioned git spawn is the
  `git-exec` adapter (a test fails on outside spawns).
- Env vars are `KANBAN_*` (or `AGENTIC_KANBAN_*`) and MUST get a row in `docs/env-vars.md`
  (`env-read-ownership.test.ts` enforces the inventory).
- Time injection uses exactly `now?` (persisted) and `nowMs` (arithmetic) — AST ratchet enforced.
- Commit subjects reference `ak-<issue#>`; never UTF-8 BOM (commit-msg hook strips it).
- New single-consumer modules in `shared/lib` fail a shrink-only ratchet; add them where consumed.
- Never: delete `kanban.db`, kill all node processes, write outside the repo from a builder.

## Verify before claiming done

```bash
pnpm typecheck && pnpm lint && pnpm lint:arch && pnpm test:unit && pnpm check
```

A green impact selection is NOT a green suite; the full suite runs only on the release
candidate (`pnpm promote --dry-run` shows the pending promotion evidence).

## Knowledge sources for deep questions (in priority order)

1. Repo `docs/decisions/001-019` (the WHY), `docs/domain/` (15 bounded contexts, DDD map), `CLAUDE.md` (rulebook).
2. graphify graph in the workspace clone: `graphify-out/graph.html`, `GRAPH_REPORT.md`, or
   `graphify query/explain/path` commands.
3. mempalace: `mempalace_search "…" wing="agentic_kanban"` (~63k drawers, rooms `packages`, `documentation`…).
4. This plugin's references and the workspace `docs/` folder.
