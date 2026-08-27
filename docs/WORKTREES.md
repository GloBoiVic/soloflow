# SoloFlow Branches

SoloFlow is branch-first. Feature and Critical work uses the normal repository checkout,
not a linked worktree.

## GIT START

Before branch creation:

```bash
git branch --show-current
```

Meaningful unrelated dirty changes block the operation. Do not silently stash, commit,
discard, reset, rebase, or switch branches.

After developer approval:

```bash
git switch -c solo/<workstream-slug>
```

Verify the branch and status, then record the branch and full base SHA in `PLAN.md` and
`ACTIVE.md`. Solo, workers, editor, dev server, Git diff, and Local Host all use this
same repository root and branch until merge or explicit abandonment.

## GIT END

After explicit merge approval, verify status and diff, commit the feature branch, switch
to `main`, verify `main`, merge the feature branch, and verify the merge. Do not silently
push, force, squash, rebase, or delete an unmerged branch.

Linked worktrees remain an explicit future technique for parallel work, not normal
SoloFlow runtime behavior.
