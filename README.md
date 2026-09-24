# AgenticKanban — ZCode plugin

A [ZCode](https://zcode.dev) plugin for **[agentic-kanban](https://github.com/p-wegner/agentic-kanban)**
— the local-first kanban board for AI-driven coding tasks (cleanroom reimplementation of
vibe-kanban). It packages everything an agent needs to **develop, operate and contribute**
to the project:

| Component | What it does |
|-----------|--------------|
| **Skill `agentickanban-dev`** | Local development: setup (Node 22 / pnpm 10), monorepo map (server / client / shared / mcp-server / desktop / e2e), mandatory conventions (`KANBAN_*` env inventory, `now?/nowMs` time injection, BOM-free `ak-<N>` commits, shrink-only ratchets), quality gates (`pnpm typecheck / lint:arch / test:unit / check`) |
| **Skill `agentickanban-contribute`** | Community workflow: sync your fork with upstream (rebase, linear history), branch naming, PR style mirrored from merged PRs, pre-PR gates |
| **Skill `agentickanban-operate`** | Board operations: npx/Docker install, the two-boards rule, `pnpm promote` release candidates, `KANBAN_*` environment variables, per-workspace Docker service stacks, worker fleet, troubleshooting |
| **Agent `agentickanban-expert`** | Expert subagent that routes any agentic-kanban question to the right skill and grounds answers in the upstream docs, decision records (001–019) and the DDD domain map |

Each skill ships a `references/` folder with distilled upstream documentation
(deployment, env vars, fleet freshness, repo state, ecosystem READMEs).

## Install (ZCode)

1. Open **Plugin Marketplace → Add → Add Plugin Marketplace**.
2. Paste this repository: `efica/agentickanban` (or clone URL
   `https://github.com/efica/agentickanban`).
3. Open **Personal → agentickanban marketplace → AgenticKanban → Install**.
4. Manage it afterwards in **Settings → Plugins**. Skills become available in the composer
   picker (`/agentickanban-dev`, `/agentickanban-contribute`, `/agentickanban-operate`);
   the `agentickanban-expert` subagent activates on agentic-kanban questions.

## Try it

> "Using agentickanban-expert: how do I sync my fork with upstream and open a PR
> referencing issue ak-1234?"

Expected: the subagent follows `agentickanban-contribute` and gives you the exact commands
(`git fetch upstream`, rebase, push to your fork, `gh pr create --repo p-wegner/agentic-kanban`).

## Credits

Knowledge compiled from the upstream
[agentic-kanban](https://github.com/p-wegner/agentic-kanban) repository (MIT) — its docs,
decision records and `CLAUDE.md` — plus public ecosystem sources (vibe-kanban by BloopAI,
Hacker News threads, community reviews). See `LICENSE`.

## License

MIT
