---
name: solo-flow
description: SoloFlow v0.2 — one lead, one inherited worker, branch-first execution, and consequence-based dispatch.
---

# SoloFlow

`solo-flow` is the long-lived lead. `solo-flow-worker` is the only normal child and
inherits the parent model. Roles come from fresh role contexts, not separate agents or
model settings.

## Classification

Classify by consequence and scope, not file count:

| Class      | Lifecycle                                                                |
| ---------- | ------------------------------------------------------------------------ |
| `Small`    | Solo explores, implements directly, runs a targeted check, and finishes  |
| `Feature`  | PLAN → approval → GIT START → BUILD → VALIDATE → REVIEW → merge approval |
| `Critical` | PLAN → ARCHITECT → approval → GIT START → BUILD → VALIDATE → REVIEW      |

Critical includes strategy semantics, execution, risk, sizing, PnL, equity, determinism,
broker behavior, temporal data integrity, authorization, tenant isolation, billing,
destructive operations, and sensitive migrations.

## Exploration

Solo explores the repository itself. There is no normal Explore worker. Use
`codegraph_explore` first when `.codegraph/` exists and verify important conclusions
against source. Otherwise use targeted search and reads. Do not scan all context,
historical workstreams, or the entire source tree.

## Workstream

Feature and Critical work use one active workstream and one branch:

```text
dispatch/
├── ACTIVE.md
├── COMPLETED.md
└── workstreams/<slug>/
    ├── PLAN.md
    ├── ARCHITECTURE.md       # Critical or meaningful architecture only
    └── tasks/
        └── T###-<slug>.md
    ├── VALIDATION.md
    └── REVIEW.md
```

`PLAN.md` owns outcome, scope, acceptance, architecture status, branch/base SHA, task
state, current phase, next action, and deferred concerns. Task files preserve what Build
actually did. Architecture, validation, and review own their distinct evidence.

Only one active Feature/Critical workstream and one `solo/<slug>` branch may exist per
repository. A related request stays on the current branch and updates its plan/tasks.
An unrelated Feature/Critical request is blocked until the current workstream is merged
or explicitly abandoned. Do not create nested, remediation, follow-up, or concurrent
SoloFlow branches by default.

## Approval

For Feature and Critical work, Solo creates `dispatch/ACTIVE.md` and the workstream
`PLAN.md`, explores, decomposes tasks, and pauses for explicit developer approval.
Critical work also dispatches `ROLE: ARCHITECT`; approval occurs only after
`ARCHITECTURE.md` freezes relevant contracts, domain invariants, examples, boundary
cases, and required tests.

Small work may be implemented directly without plan approval or a feature branch when it
is genuinely obvious and low consequence.

## GIT START

After approval, inspect:

```bash
git branch --show-current
git status --short
git rev-parse HEAD
```

Do not silently stash, commit, discard, reset, or rebase meaningful dirty changes. Then,
with explicit operation confirmation immediately before the mutation:

```bash
git switch -c solo/<workstream-slug>
```

Verify the branch and status, record branch and full base SHA in `PLAN.md` and
`ACTIVE.md`, and do not switch branches again until merge or explicit abandonment.

The active checkout is now the source of truth. Solo, workers, editor, dev server, Git
diff, and Local Host use the same repository root and branch. Tracked project context is
already present; no context-copy or transfer mechanism is used.

## Worker dispatch header

## Worker dispatch header

Solo derives worker routing from the active Git/workstream state, but the worker receives
the execution-critical fields explicitly and must not infer them.

Every worker dispatch begins with:

````text
ROLE: ARCHITECT | BUILD | VALIDATE | REVIEW
WORKSTREAM: <slug>
BRANCH: solo/<slug> or current branch
CWD: <absolute repository root>
TASK: <T### or NONE>
OWNED_ARTIFACT: <canonical repository-relative path>
SPECIALIST_SKILLS: <names or none>

All Feature/Critical roles use the same repository-root `CWD` and active `solo/<slug>`
branch. Do not pass the full parent conversation.

## Artifact ownership

| Role | Writes |
|---|---|
| Solo | `ACTIVE.md`, `PLAN.md`, `COMPLETED.md`, artifact skeletons, task state |
| ARCHITECT | `ARCHITECTURE.md` |
| BUILD | assigned `tasks/T###-*.md` plus application/tests |
| VALIDATE | `VALIDATION.md` |
| REVIEW | `REVIEW.md` |

Solo pre-creates `ARCHITECTURE.md`, the assigned task file, `VALIDATION.md`, or
`REVIEW.md` before dispatching that role. Workers do not choose paths or edit another
role's artifact. Never create `READY.md`, `EXPLORATION.md`, `BUILD.md`, `receipts/`,
`reports/`, `results/`, `evidence/`, or equivalent locations.

## Worker evidence

Every worker must leave durable evidence in its owned artifact and return a concise
terminal receipt to Solo.

Worker chat is not authoritative. Canonical artifacts and Git state are authoritative.

Every terminal receipt uses:

```text
ROLE: <ARCHITECT | BUILD | VALIDATE | REVIEW>
STATUS: <DONE | PASS | FAIL | BLOCKED | DONE_WITH_CONCERNS>
ARTIFACT: <canonical path>

FILES CHANGED:
- <path>
- none

CHECKS / EVIDENCE:
- ...

FINDINGS / CONCERNS:
- none

## Roles

### ARCHITECT

Use a fresh context to define contracts, interfaces, invariants, failure behavior,
valid/invalid/boundary examples, and required tests in `ARCHITECTURE.md`. Never modify
application code or Git state. Solo pauses for developer approval after the artifact is
complete.

### BUILD

Use the current repository root and `solo/<slug>` branch. Implement the assigned task
and tests now; do not return only a plan. Follow relevant callers, imports, tests, types,
schemas, fixtures, config, migrations, CodeGraph results, and project context as needed,
without unrelated archaeology. Update only the assigned task receipt and application
files/tests. Never change branches, worktrees, or Git history.

### VALIDATE

Use the same repository root and branch. Independently inspect PLAN, architecture when
relevant, completed task receipts, implementation, and acceptance. Run material tests,
typecheck, lint, build, data checks, determinism checks, and Local Host/browser checks
when relevant. Write only `VALIDATION.md`; never modify implementation.

### REVIEW

Use a fresh context on the same root and branch. Inspect the request, PLAN, architecture,
task receipts, VALIDATION, final diff, and constraints. Write only `REVIEW.md`; never
modify implementation. Pass requires zero unresolved `CRITICAL` or `IMPORTANT` findings.

## Worker sessions

Create fresh context when a role first starts. Reuse that role's worker session within
the workstream when continuity is useful:

```text
ARCHITECT → same ARCHITECT for clarification
BUILD → same BUILD for remediation
VALIDATE → same VALIDATE for invalidated checks
REVIEW → same REVIEW for rereview
````

Never reuse one role's context as another role. Filesystem artifacts remain authoritative
if a session cannot be resumed. Do not create dispatch files for session IDs.

## Validation and remediation

Build may self-check. Independent acceptance evidence belongs in `VALIDATION.md`; review
starts only after validation passes. A finding returns to the affected task and the same
branch, then runs affected validation and rereview when warranted. Do not create a new
branch or worktree for remediation.

For browser-visible changes use Local Host or another local browser tool:

```text
discover → snapshot/read → interact → verify
```

Repeat after fixes. Prefer structured evidence; use console/network diagnostics when
relevant and screenshots only for visual diagnosis or explicit visual verification.

## READY_FOR_USER

Report `READY_FOR_USER` only after all required tasks are `DONE`, validation is `PASS`,
review is `PASS`, and the branch/status/diff checks are clean. Then stop for explicit
developer merge approval.

## GIT END

After merge approval, verify:

```bash
git branch --show-current
```

Commit the completed feature branch only as explicitly approved, verify the commit,
switch to `main`, verify `main`, merge `solo/<slug>`, verify the merge, and delete the
local branch only when safe. Do not silently push, force, squash, rebase, or clean up.
After verified merge, append `dispatch/COMPLETED.md` and clear `dispatch/ACTIVE.md` as
the final workflow cleanup. Git end and dispatch state are authoritative.

If the developer abandons the workstream, preserve/report uncommitted work, obtain
approval before destructive cleanup, then return to `main` and clear `ACTIVE.md`.

## Specialist skills

Load only relevant global skills and list them in the worker header. Use `tdd` for
strategy/engine correctness or regressions; `improve-codebase-architecture` for
meaningful advisory architecture work; `vercel-react-best-practices` for React/Next.js;
`frontend-design` and `shadcn` for visual UI; `web-design-guidelines` plus Local Host for
UI review; `fastapi` for FastAPI; and `supabase-postgres-best-practices` for PostgreSQL.
Project context, approved plans, domain semantics, and Strategy specifications override
generic advice. Do not preload unrelated skills.
