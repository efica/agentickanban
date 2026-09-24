# Developing on agentic-kanban — Community Contributor Guide

> Compiled 2026-09-24 from the upstream repository (p-wegner/agentic-kanban), its docs,
> and ecosystem research. Paths like `docs/...` refer to files inside the repository.

## 1. What this project is

**agentic-kanban** is a local-first, single-user kanban board for managing AI-driven coding
tasks — a **cleanroom reimplementation of [vibe-kanban](web/vibe-kanban-readme-upstream.md)**
(BloopAI). The original was sunsetted by Bloop in April 2026 (see the ["Goodbye Bloop"
announcement](https://vibekanban.com/blog/goodbye-bloop), discussed on
[HN](https://news.ycombinator.com/item?id=47718190)); this project keeps the useful core —
the pipeline connecting coding agents (Claude Code, Codex, Copilot, Pi) to a board via
MCP → REST API → SQLite — while dropping cloud/multi-tenant/OAuth features in favor of
**testability** and simplicity.

- License: MIT · Language: TypeScript (pnpm monorepo) · Author: Peter Wegner (also author of
  the [Coding-Aider](web/coding-aider-readme.md) JetBrains plugin)
- The project is heavily **dogfooded**: it is developed using itself, which explains its
  unusual emphasis on guards, ratchets, and evidence-based gates.

## 2. Setup

Prerequisites: **Node LTS 22** (Node 23 hangs under `tsx watch`), **pnpm 10** (workspace),
Docker optional (service stacks per workspace).

```bash
pnpm install
pnpm db:setup        # migrate + seed + register this repo as a project
pnpm dev             # server :3001 + client :5173
```

Gotchas for clean clones are collected in the repo's `docs/install.md` (pnpm ENOENT from
stale state, shared-store resolution, boot-from-dist smoke test). Prefer `npx
agentic-kanban dev` / the Docker image (`pwegner3141/agentic-kanban`) if you don't want to
build from source.

## 3. Architecture (pnpm workspace)

| Package | Role |
|---------|------|
| `packages/server` | REST API + WebSocket board events, agent execution, workspaces (git worktrees), merge queue/train, risk posture, worker fleet |
| `packages/client` | React + Vite board UI (optimistic mutations, live reconciliation) |
| `packages/shared` | Shared types/schema, single-source-of-truth services (git-service, profile roster) |
| `packages/mcp-server` | MCP server (~35 tools) exposing the board to coding agents |
| `packages/desktop` | Tauri v2 desktop wrapper |
| `packages/e2e` | Playwright E2E suites (the project had E2E from day one) |

**Load-bearing design decisions** (each documented in `docs/decisions/001..019`, distilled in
`docs/domain/`):

- **Two boards** — the dev board you talk to is not the stable board you operate; promotion
  happens via timed `pnpm promote` against a green full-suite sweep (decision 019), never per-merge.
- **Git single source of truth** — all high-level git goes through one `git-service.ts`; the only
  sanctioned way to spawn git is the `git-exec` adapter (a test fails on any outside spawn).
- **Per-workspace Docker stacks** — deterministic compose project names, create-time free ports,
  per-board-instance scoping (decision 011).
- **Risk posture dial** (`strict|standard|fast|sprint|iterate|flow`) — one named preference per
  project replaces ~8 hand-aligned prefs; a weaker posture may only weaken verification *visibly*
  (decision 017).
- **Worker Fleet** — pull model: remote workers dial the board; landing is fast-forward-only into
  `refs/kanban/incoming`, bound to assignment tokens; credentials never leave the worker machine
  (decision 012).
- **Ratchets everywhere** — coverage floors, single-consumer `shared/lib`, always-run gate list,
  dependency pinning: shrink-only sets that fail CI on regression.

## 4. Conventions you must follow (from `CLAUDE.md` / `AGENTS.md`)

- `CLAUDE.md` is the canonical agent instruction file; read it before changing anything.
- **Env vars** are prefixed `KANBAN_*` and every `process.env` read must be registered in
  `docs/env-vars.md` (an inventory gate enforces it).
- **Time injection** uses exactly two spellings: `now?` (persisted timestamps) and `nowMs`
  (arithmetic); an AST-shape ratchet keeps it that way.
- **Commit messages**: subject pattern `ak-<issue#>: ...`; never with UTF-8 BOM (Windows
  PowerShell footgun, enforced by commit-msg hooks).
- **Scope discipline**: change only what the task requires; unrelated fixes become tickets.
- **Never** delete `kanban.db`, never kill all node processes, never write to unrelated repos
  (cross-worktree write guard).

## 5. Quality gates before you open a PR

```bash
pnpm typecheck        # plus :shared/:server/:mcp variants
pnpm lint             # and pnpm lint:arch for architecture boundaries
pnpm test:unit        # targeted suites; merge gate runs scoped impact selection
pnpm check            # the combined pre-merge check
```

The merge gate runs a scoped test-impact selection plus the unconditional `@gate:always-run`
floor; coverage is push-only and informational (per-package ratchet floors). A green *impact
selection* is explicitly **not** a green suite — expect the release-candidate sweep to run the
full suite on a separate branch.

## 6. Contribution workflow

1. Sync your fork: `git fetch upstream && git rebase upstream/master` (linear history).
2. Branch per change; issues reference `ak-<N>`. The repo has no CONTRIBUTING.md yet — mirror
   the style of [recent merged PRs](repo/merged-prs-recent.md) (small, ticket-referenced, tests included).
3. PRs target `p-wegner/agentic-kanban` `master`. There are no GitHub Discussions or wiki —
   GitHub issues are the only channel (most issues are closed quickly; see
   [open](repo/open-issues.md) / [recent closed](repo/closed-issues-recent.md)).

## 7. Where the knowledge lives (repo docs map)

- `docs/decisions/001-019` — architecture decision records (the "why").
- `docs/domain/` — 15 bounded contexts with DDD context map (the "what").
- `docs/prd/` — product requirements (data model, agent integration, MVP scope).
- `docs/analysis/`, `docs/plans/`, `docs/verification/` — measurements, plans, test strategy.
- `docs/learnings/` — post-incident learnings (each guard in the code is a fossil of one).
- `CLAUDE.md` — the operational rulebook distilled.

## 8. Ecosystem & background reading (in this folder)

- [`web/vibe-kanban-readme-upstream.md`](web/vibe-kanban-readme-upstream.md) — the original
  project being reimplemented (sunsetting notice included).
- [`web/vibe-guide.md`](web/vibe-guide.md) — Bloop's tips & best practices for AI-coding-agent workflows.
- [`web/hn-vibe-kanban-show-thread.md`](web/hn-vibe-kanban-show-thread.md) — launch discussion (195 pts).
- [`web/hn-goodbye-bloop-thread.md`](web/hn-goodbye-bloop-thread.md) — shutdown announcement pointer.
- [`web/virtuslab-vibe-kanban-review.md`](web/virtuslab-vibe-kanban-review.md) — third-party review of the original.
- [`web/heym-loop-engineering.md`](web/heym-loop-engineering.md) and
  [`web/loop-engineering-osmani.md`](web/loop-engineering-osmani.md) — the "loop engineering"
  concepts behind agentic kanban boards.
- [`web/coding-aider-readme.md`](web/coding-aider-readme.md) — the author's previous AI-coding tool.
- [`web/vibe-kanban-alternative-mem0-readme.md`](web/vibe-kanban-alternative-mem0-readme.md) —
  another community alternative (persistent-memory angle).
