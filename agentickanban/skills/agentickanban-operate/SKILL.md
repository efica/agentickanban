---
name: agentickanban-operate
description: Operate a deployed agentic-kanban board — install via Docker/npx, two-boards mode, env vars, promotion (pnpm promote), worker fleet, per-workspace service stacks and troubleshooting. Use when the user wants to install, configure, promote, monitor or troubleshoot a running board.
---

# Operate agentic-kanban

## Install (pick one)

```bash
npx agentic-kanban dev                       # published package, no build
docker pull pwegner3141/agentic-kanban:latest  # Docker Compose setup in repo docs/deployment.md
# or from source: pnpm install && pnpm db:setup && pnpm dev (server :3001, client :5173)
```

## The two-boards rule (most important operational fact)

The board you TALK to (dev) is not the board you OPERATE (stable). Landing code on master
does not change the operated board: promotion moves a green release candidate to the stable
board on a cadence — never per merge.

- `KANBAN_BOARD_ROLE` decides ports AND database together (dev board never touches the
  operated DB — split-brain prevention).
- Promotion: `pnpm promote` (scheduled on cadence; `--dry-run` shows which rc, which sweep
  row and its evidence). A red smoke test RETIRES the `stable-<date>` tag name.
- Release candidate flow: cut `rc/<date>` from master → full-suite sweep on the rc → green:
  tag `stable-<date>`, deploy (build, migrate, restart, smoke), merge back. Red: file heal
  tickets ON the rc, never on master (decision 019).

## Environment variables

Board-owned vars are `KANBAN_*` (legacy bare names still work via `readBoardEnv` with a
deprecation log). Key ones: `KANBAN_DB_URL` (explicit libsql URL, wins over every
DB-location rule), `KANBAN_BOARD_ROLE`, `KANBAN_FLEET_PORT` / `KANBAN_GIT_HTTP_PORT`
(opt-in fleet listeners — never mount the board API on them), `KANBAN_STACK_PORT_RANGE`
(published ports for per-workspace compose stacks), `KANBAN_TLS_CERT`/`KANBAN_TLS_KEY`
(HTTP/2 — multiplexes past the browser 6-connections-per-origin cap). The complete
inventory is `docs/env-vars.md` and a test enforces every `process.env` read is documented.

## Docker service stacks

Each workspace gets its own compose stack: deterministic project name keyed by workspace id,
create-time free host ports, per-board-instance scoping so two boards sharing a daemon never
reap each other's stacks. Wide-sweep GC is operator-driven only. DooD (host socket) gives
agents host-root-equivalent power and bypasses PreToolUse hooks — trusted isolated hosts
only; DinD sidecar for containerized boards. Graceful degradation: no Docker → the
no-docker workflow is unchanged.

## Worker Fleet (remote compute)

Pull model: workers dial the board (NAT-friendly; pair with board URL + token). Landing is
fast-forward-only via `refs/kanban/incoming` bound to assignment tokens; diverged = held,
never force-updated. Credentials never leave the worker machine (allowlist projection).
Version freshness: protocol handshake 409 on mismatch; there is still NO auto-update
mechanism — rebuild the worker tarball by hand (see `docs/fleet-version-freshness.md`).

## Troubleshooting pointers

- Boot/ports/DIND: repo `docs/deployment.md`, `docs/env-vars.md` (port ladders).
- Merge train stuck/red-base: `docs/decisions/019`, `docs/analysis/2026-09-16-merge-throughput-blockers.md`.
- Vite dev proxy exposed API: `VITE_HOST=127.0.0.1` is the emergency exit (loopback guard).
- Deep history: mempalace `wing="agentic_kanban"`; graph: `graphify-out/graph.html`.
