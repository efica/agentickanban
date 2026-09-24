# vibe-guide — summary (third-party source, not redistributed verbatim)

> Source: https://www.vibekanban.com/vibe-guide — "Vibe Guide: tips and best practices
> for working with AI coding agents" (BloopAI). The original page text is copyrighted
> by its author and is NOT covered by this repository's MIT license, so only this
> original summary is bundled. Read the source for the full text.
>
> Notable: the page also carries the "Vibe Kanban is sunsetting" notice — the project
> continues as open source, community maintained.

The guide is organized as short, opinionated practices for orchestrating coding agents:

- **Plan first**: always start from a plan, however small the task; have the agent propose
  it and confirm before making changes.
- **Plan more, review less**: minutes spent planning save more in review — a plan is
  faster to judge than a diff.
- **Small, reviewable tasks**: keep each agent task narrow enough that its diff can be
  reviewed in one sitting; decompose big tasks before spawning.
- **Isolated workspaces**: let each agent work in its own git worktree so parallel agents
  never fight over the same checkout.
- **Inspect while they run**: watch agent activity and steer early instead of waiting for
  a finished (wrong) result; kill and re-prompt cheaply.
- **Review the diff, not the promise**: judge the work by the produced diff against the
  base branch and run the project's own verification before accepting.

These practices map almost one-to-one onto how agentic-kanban implements boards,
workspaces, agents and the review/merge flow (see this plugin's `agentickanban-dev`
and `agentickanban-operate` skills).
