---
name: agentickanban-contribute
description: Contribute to p-wegner/agentic-kanban from a fork — sync with upstream, branch from the development line, respect house conventions, pass quality gates and open PRs. Use when the user wants to file issues, open pull requests or sync the fork with upstream.
---

# Contribute to agentic-kanban upstream

Upstream: `https://github.com/p-wegner/agentic-kanban` (branch `master`). Expected fork
remotes in the local clone: `origin` = your fork (push here), `upstream` = p-wegner (pull /
PR target). Upstream state snapshot: [references/upstream-state.md](references/upstream-state.md).

## 1. Sync the fork (linear history, no merge bubbles)

```bash
git fetch origin && git fetch upstream
git rebase upstream/master            # or rebase your branch onto it
```

Never `git push --force`; if needed use `--force-with-lease` only with explicit user approval.

## 2. Branch and work

- Branch from `dev-efica` (local development line) or directly from `upstream/master` for
  cleanest PRs; one logical change per commit, descriptive messages referencing `ak-<N>`.
- House style for PRs (match the merged ones): small scope, ticket-referenced subject,
  tests included, no unrelated fixes (scope discipline — unrelated fixes become tickets).
- There is NO CONTRIBUTING.md, wiki or GitHub Discussions upstream: GitHub issues are the
  only channel. `CLAUDE.md` in the repo is the rulebook.

## 3. Gates before opening the PR

```bash
pnpm typecheck && pnpm lint:arch && pnpm test:unit && pnpm check
```

Plus the convention ratchets (they run in CI): env-var inventory, time-injection spellings,
BOM-free commits, `shared/lib` single-consumer ratchet, always-run gate list.

## 4. Open the PR

```bash
git push origin <branch>            # to YOUR fork only
gh pr create --repo p-wegner/agentic-kanban --base master --head <owner>:<branch>
```

Useful upstream context commands: `gh issue list --repo p-wegner/agentic-kanban`,
`gh pr list --repo p-wegner/agentic-kanban --state merged` (style reference).

## 5. After merge

`git fetch upstream && git branch --set-upstream-to=upstream/master` on the line, rebase,
delete the feature branch, and pull the change into `dev-efica`.
