# SoloFlow Worktrees

Feature and Critical implementation normally uses a linked worktree. Solo dispatches the
generic worker with `ROLE: WORKTREE`; there is no separate Worktrees agent and no
`READY.md` artifact.

Before creation, inspect:

```bash
git status --short
```

Use:

```text
branch: solo/<workstream-slug>
worktree: ../<repo>--wt--<workstream-slug>
```

Record the full base SHA, branch, and path in `PLAN.md`. Confirm each exact
repository-changing command immediately before running it. Never silently stash,
discard, commit, push, merge, rebase, reset, switch branches, or remove worktrees.
The WORKTREE worker returns the exact cwd, branch, base SHA, context status, and blockers.
BUILD, VALIDATE, and REVIEW then use that same cwd. Preserve the worktree for
`READY_FOR_USER` inspection.
