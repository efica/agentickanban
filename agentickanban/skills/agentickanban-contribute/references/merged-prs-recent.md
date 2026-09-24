# PR #7: Integrate #996-#1003 fork/workflow work with 1319 upstream commits
Mergeado: 2026-08-20T15:42:30Z | Autor: p-wegner | Rama: integrate/fork-workflow-996-1003
Integrates the 31 unpushed local commits (fixes **#996-#1003**: fork-child guards, workflow status-sync, `--dry-run` cleanup flag, provider logging) with the 1319 upstream commits on `master`. Merge-base was `91f3eb1c`.

Pushed in two steps on purpose:

| sha | what |
|---|---|
| `a7aa4222` | the pre-merge state, pushed as-is so the local-only work exists on the remote before any resolution |
| `68fe2220` | `git merge origin/master` with the 10 conflicts resolved |
| `7d2878da` | two local test fixtures re-pointed at module seams upstream moved (see below) |

**Not rebased** — the 31 commits include 7 merge commits. `rerere` was enabled and recorded preimages but auto-resolved none of the 10.

## Per-file resolution

| File | Kind | Resolution |
|---|---|---|
| `.claude/hooks/validate-command-safety.js` | **mechanical, but by hand** | No 3-way merge available (see below). Took upstream wholesale, re-applied local's additive block. |
| `packages/mcp-server/src/tools/mark-ready-for-merge.ts` | mechanical | Kept upstream's injected `ToolDeps` + `mcpJson` shape; added `mcpStructuredError` to the `db-utils` import. Local's #1001 fork-child guard and amended tool description had already auto-merged in the body. |
| `packages/server/src/__tests__/helpers/test-db.ts` | mechanical | Upstream's extracted `readMigrationStatements(file, MIGRATIONS_DIR)`; kept local's `const tag` + `__drizzle_migrations` stamping. |
| `packages/server/src/__tests__/stranded-review-reconciler-relaunch.test.ts` | mechanical | Union — both sides' `it(...)` blocks kept (local's #998 fork-child case, upstream's #270 merge-in-flight and review-launch-pending cases). 6 tests total. |
| `packages/server/src/cli/commands/project.ts` | mostly mechanical | Upstream's rewrite (`cliAction()` wrapper, temp-fixture cleanup, no explicit `runMigrations()`); re-applied local #1002's `--dry-run` option, merged both description texts. **Judgement call:** the early return sits *before* upstream's temp-fixture unregi

---

# PR #8: feat(worker): ship the Windows fleet-worker service + tray in the npm package
Mergeado: 2026-08-20T15:40:09Z | Autor: p-wegner | Rama: feat/worker-windows-service
Makes the Windows worker tooling part of this repo and retrievable from the published package, so a worker machine gets it from `npm i -g agentic-kanban` (or a `scripts/pack-worker.mjs` tarball) instead of someone hand-copying five scripts out of a local `C:\Tools`.

## What lands

`packages/server/tools/worker-windows/` — chosen because `packages/server` already ships ancillary asset directories (`plugins/`, `skills/`) at its package root and lists them in `files`, and `plugins/app-runner/tools/` establishes `tools` as the name for executable helpers:

| Script | Role |
|---|---|
| `ak-worker.ps1` | install / replace / remove the worker tarball; sha256 verify, refuses a tarball whose bin map lacks `agentic-kanban-worker`, compares the installed manifest afterwards to catch an npm same-version cache hit. Never touches `~/.agentic-kanban`. |
| `ak-worker-service.ps1` | `-Install/-Uninstall/-Start/-Stop/-Restart/-Status/-Log`; registers the Scheduled Task `AgenticKanbanWorker` at logon in the USER session (agents must run with this user's provider credentials, so not a SYSTEM service). |
| `ak-worker-run.ps1` | the supervised wrapper the task runs: sets `ACP_AUTOCONNECT=0` and `CLAUDE_CONFIG_DIR` explicitly, strips inherited `CLAUDE_*` session vars, resolves the worker bin (a task's PATH lacks the npm global bin), timestamps every daemon line into `%LOCALAPPDATA%\agentic-kanban-worker\worker.log`, restarts with backoff. |
| `ak-worker-tray.ps1` | WinForms NotifyIcon, grey/red/yellow/green/blue; state from the log tail + process check, board `/api/health` on a slower timer, single-instance mutex. |
| `ak-worker-tray-launch.vbs` | hidden launcher. |

Copied verbatim from the machine they were verified on — their comments carry the reasoning (the `ACP_AUTOCONNECT` inheritance that wedged two fleet dispatches, the account-selection hazard of an unset `CLAUDE_CONFIG_DIR`, the npm same-version cache trap, the tray GDI-leak and `Forms.Timer` conventions) and are deliberately

---

# PR #6: fix(worker): the disconnect log says WHY the socket went away
Mergeado: 2026-08-20T15:38:20Z | Autor: p-wegner | Rama: fix/worker-close-code
## Problem

The worker daemon's WebSocket `close` handler took no arguments:

```ts
socket.on("close", () => {
  if (ws === socket) ws = null;
  scheduleReconnect();
});
```

Both the close **code** and the close **reason** were discarded, so every way a worker can lose its socket rendered as the same log line. In particular a deliberate board-side close was indistinguishable from a dead transport:

- `worker-connection.service.ts` `handleOpen()` evicts the previous socket for the same `workerId` when a second connection arrives (`existing.ws.close()`), and `closeConnection()` does the same on revocation. Both are bare `close()` calls — a clean `1000`/`1005` with no reason.
- A `stop` from the board is likewise a clean close.
- A network path going away or the board process being killed is `1006` — the socket broke with **no close frame at all**.

These want opposite responses (reconnect vs. stop reconnecting / investigate why the board is evicting us), and a self-sustaining reconnect→evict loop looked exactly like a flaky link.

The old line made this worse rather than merely unhelpful:

```
[worker] disconnected; retrying in 1s
```

The only number it printed was the **retry backoff delay** — and `reconnectDelay` is reset to `RECONNECT_MIN_MS` inside the `open` handler. So after any successful connection the next disconnect always reported `1s`, regardless of whether the socket had been up for two seconds or two hours. A log of bare `retrying in 1s` lines reads like a socket dying instantly every time. **This actively misled a live debugging session today.**

## Fix

The handler now takes `(code: number, reason: Buffer)` and the log line reports why the socket went away and how long it had been up:

```
[worker] disconnected (code 1006 - transport failed, no close frame; up 3812s); retrying in 1s
[worker] disconnected (code 4001: evicted by a newer connection - closed by board; up 4s); retrying in 1s
```

- `code 1006` is named `transport failed, no close frame`; 

---

