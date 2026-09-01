# SoloFlow Dispatch

SoloFlow uses one active workstream and one `solo/<slug>` branch per repository.

## Lifecycle

```text
Feature/Critical request → classify → Solo explores → PLAN (no BUILD task)
→ approval → git switch -c solo/<slug> → create tasks/T001 → BUILD → VALIDATE → REVIEW
→ READY_FOR_USER → merge approval → GIT END → close dispatch

Small request → classify → Solo explores → implement → targeted check → done
```

Small, obvious changes may stay on the current branch. Feature and Critical work create
`dispatch/ACTIVE.md` and planning artifacts before implementation, but no `tasks/` directory,
BUILD assignment, or `READY` task before explicit developer approval. Critical work also
requires `ARCHITECTURE.md` with frozen domain invariants before the separate developer
implementation approval gate.

## Structure

```text
dispatch/
├── ACTIVE.md
├── COMPLETED.md
└── workstreams/<slug>/
    ├── PLAN.md
    ├── ARCHITECTURE.md       # Critical or meaningful architecture only
    ├── tasks/T###-<slug>.md       # after approval and GIT START
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

Before approval, Feature normally contains only `PLAN.md`; Critical contains `PLAN.md` and
`ARCHITECTURE.md`. Feature uses `PLAN_PENDING_APPROVAL`; Critical uses
`DEVELOPER_APPROVAL`. Feedback revises planning artifacts but is not approval. After explicit
approval, Solo records it, performs and verifies GIT START, then creates `tasks/` and the
first `T001` BUILD assignment with `Status: READY`.

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

PLAN or ARCHITECTURE completion alone never authorizes BUILD dispatch. If an accidental
pre-approval task exists, do not build from it; restore the approval phase, preserve any
execution evidence as historical unauthorized work, and create the active assignment only
after approval and GIT START.

## Git end

After the latest validation and review chain pass, Solo reports `READY_FOR_USER` and waits
for merge approval. Solo then verifies the branch/status/diff, commits the feature branch,
switches to `main`, merges `solo/<slug>`, verifies the merge, and safely cleans up only
as explicitly approved. Push is never implicit. Finally append `COMPLETED.md`, clear
`ACTIVE.md`. Git end and dispatch state are the source of truth for cleanup.
