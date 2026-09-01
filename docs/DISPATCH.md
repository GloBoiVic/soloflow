# SoloFlow Dispatch

SoloFlow uses one active workstream and one `solo/<slug>` branch per repository.

## Lifecycle

```text
request → classify → Solo explores → PLAN → approval when required
→ git switch -c solo/<slug> → BUILD → VALIDATE → REVIEW
→ READY_FOR_USER → merge approval → GIT END → close dispatch
```

Small, obvious changes may stay on the current branch. Feature and Critical work create
`dispatch/ACTIVE.md` and `dispatch/workstreams/<slug>/PLAN.md` before implementation.
Critical work also requires approved `ARCHITECTURE.md` and domain invariants.

## Structure

```text
dispatch/
├── ACTIVE.md
├── COMPLETED.md
└── workstreams/<slug>/
    ├── PLAN.md
    ├── ARCHITECTURE.md       # Critical or meaningful architecture only
    ├── tasks/T###-<slug>.md
    ├── VALIDATION.md
    ├── REVIEW.md
    └── remediations/R###-<slug>/
        ├── BUILD.md
        ├── VALIDATION.md
        └── REVIEW.md
```

`PLAN.md` owns scope, acceptance, branch/base SHA, task state, current phase, next action,
and concerns. Completed task, validation, review, and remediation files are immutable
evidence. Each role owns only its artifact; downstream artifacts reference receipts rather
than copy or overwrite them.

## Remediation

An approved-scope defect creates the next sequential `remediations/R###-<slug>/` chain:

```text
finding → BUILD.md → VALIDATION.md → REVIEW.md
```

Do not reopen an original task or overwrite root evidence. A remediation BUILD packet names
the origin finding, severity, related task, violated requirement or invariant, exact scope,
out-of-scope items, and required regression evidence. A new requirement or architecture
expansion returns to its approval gate instead.

Validation or review findings create another remediation directory; earlier receipts remain
unchanged. The existing workstream-wide remediation cap and same-worker BUILD reuse rules
still apply, but reuse never means reusing an artifact.

## One active branch

Before new Feature/Critical work, inspect `git branch --show-current`, `git status --short`,
and `dispatch/ACTIVE.md`. A related request continues on the current branch. An unrelated
request is blocked until the active workstream is merged or explicitly abandoned.

## Git end

After the latest validation and review chain pass, Solo reports `READY_FOR_USER` and waits
for merge approval. Solo then verifies the branch/status/diff, commits the feature branch,
switches to `main`, merges `solo/<slug>`, verifies the merge, and safely cleans up only
as explicitly approved. Push is never implicit. Finally append `COMPLETED.md`, clear
`ACTIVE.md`. Git end and dispatch state are the source of truth for cleanup.
