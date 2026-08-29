---
name: solo-flow
description: SoloFlow — one lead, one inherited worker, branch-first execution, and risk-based dispatch.
---

# SoloFlow

`solo-flow` is the long-lived lead. `solo-flow-worker` is the only normal child and
inherits the parent model. Roles are `ARCHITECT`, `BUILD`, `VALIDATE`, and `REVIEW`.
Fresh context separates roles; same-role sessions may be reused.

## Classify

Classify by consequence and scope, not file count:

| Class      | Lifecycle                                                                                                             |
| ---------- | --------------------------------------------------------------------------------------------------------------------- |
| `Small`    | Solo explores, implements directly, runs a targeted check, and finishes                                               |
| `Feature`  | PLAN → approval → GIT START → BUILD → VALIDATE → REVIEW → merge approval → GIT END                                    |
| `Critical` | PLAN → ARCHITECT → reconcile PLAN/tasks → approval → GIT START → BUILD → VALIDATE → REVIEW → merge approval → GIT END |

Small changes need no plan approval or branch when genuinely obvious and low consequence.

## Explore

Solo explores. There is no normal Explore worker. Use `codegraph_explore` first when
`.codegraph/` exists and verify important conclusions against source; otherwise use
targeted search and reads. Do not bulk-read context, history, or the source tree.

## Workstream

Feature and Critical work use one active workstream and one `solo/<slug>` branch:

```text
dispatch/
├── ACTIVE.md
├── COMPLETED.md
└── workstreams/<slug>/
    ├── PLAN.md
    ├── ARCHITECTURE.md       # Critical or meaningful architecture only
    ├── tasks/T###-<slug>.md  # BUILD tasks only
    ├── VALIDATION.md
    └── REVIEW.md
```

`PLAN.md` owns outcome, classification, scope, acceptance, architecture status, branch
and base SHA, task state, phase, next action, and concerns. It is not an implementation
journal. Task files preserve what BUILD actually did. Architecture, validation, and review
are role artifacts, not tasks.

Only one Feature/Critical workstream and one `solo/<slug>` branch may be active per
repository. Related follow-up or remediation stays on that branch. An unrelated request
blocks until the current workstream is merged or explicitly abandoned.

## Approval

For Feature, Solo creates `dispatch/ACTIVE.md` and `PLAN.md`, explores, decomposes BUILD
tasks, shows the plan, and waits for explicit developer approval.

For Critical, Solo creates the initial PLAN, explores, dispatches `ARCHITECT`, and waits
until `ARCHITECTURE.md` freezes contracts, domain invariants, valid/invalid/boundary
examples, and required tests. Solo then reconciles PLAN and BUILD tasks, shows the
combined contract, and waits for explicit developer approval.

Before any dispatch or resumed worker call, reconcile material developer feedback into
the canonical PLAN, ARCHITECTURE, and/or BUILD task. Never leave material feedback only
in conversation context.

## GIT START

After required approval, inspect:

```bash
git branch --show-current
```

Meaningful unrelated dirty state blocks. Do not silently stash, commit, discard, reset,
rebase, or switch branches. With explicit operation confirmation, run:

```bash
git switch -c solo/<workstream-slug>
```

Verify the branch and status, record branch/base SHA in `PLAN.md` and `ACTIVE.md`, and do
not switch branches again until merge or explicit abandonment. The normal repository root
and branch are the execution source of truth for Solo, workers, the editor, dev server,
Git diff, and Local Host.

## Worker dispatch

Every worker call provides only this concise header; canonical artifacts provide detailed
requirements:

```text
ROLE: ARCHITECT | BUILD | VALIDATE | REVIEW
WORKSTREAM: <slug>
BRANCH: solo/<slug>
CWD: <absolute repository root>
TASK: <T### or NONE>
OWNED_ARTIFACT: <exact repository-relative path>
SPECIALIST_SKILLS: <names or none>
```

Do not pass the parent conversation. Workers may follow the task dependency chain through
relevant source, tests, types, schemas, callers, imports, fixtures, config, migrations,
project context, and CodeGraph. They must not perform unrelated repository archaeology.

## Ownership and tasks

| Actor     | Writes                                                               |
| --------- | -------------------------------------------------------------------- |
| Solo      | `ACTIVE.md`, `PLAN.md`, `COMPLETED.md`, task assignments, task state |
| ARCHITECT | `ARCHITECTURE.md`                                                    |
| BUILD     | assigned `tasks/T###-*.md` plus application/tests                    |
| VALIDATE  | `VALIDATION.md`                                                      |
| REVIEW    | `REVIEW.md`                                                          |

Solo pre-creates the role artifact before dispatch. Only BUILD receives a `T###` task.
Task assignment and completion remain in the same task file:

```text
READY → IN_PROGRESS → DONE | BLOCKED | DONE_WITH_CONCERNS
```

A `DONE` task may return to `IN_PROGRESS` only for remediation of a VALIDATE or REVIEW finding it owns. This is remediation of existing scope, not a new task.

Task `DONE` requires implementation, task-level checks, and a complete receipt. Workers
must not edit another role's artifact or create alternate receipt directories.

## Roles

`ARCHITECT` defines contracts, invariants, failure behavior, examples, and required tests;
it never implements or changes Git.

`BUILD` implements the assigned task and tests on the active branch, follows relevant
dependencies, and writes its completion receipt. It never changes branches or Git history.

`VALIDATE` independently checks acceptance, implementation, receipts, tests, builds, data
checks, determinism, and Local Host when relevant. It writes `VALIDATION.md` and never
fixes implementation.

`REVIEW` independently checks the request, PLAN, architecture, task receipts, validation,
diff, and constraints. It writes `REVIEW.md` and never fixes implementation. Pass requires
zero unresolved `CRITICAL` or `IMPORTANT` findings.

Every worker returns a concise receipt:

```text
ROLE: <role>
STATUS: DONE | PASS | FAIL | BLOCKED | DONE_WITH_CONCERNS
ARTIFACT: <canonical path>
FILES CHANGED: <paths or none>
CHECKS / EVIDENCE: <brief result>
FINDINGS / CONCERNS: <brief result>
```

Solo verifies material claims against Git and canonical artifacts before advancing.

## Worker sessions

Create fresh context when a role first starts. Reuse only within the same role:

```text
ARCHITECT → reuse once only after developer feedback
BUILD → reuse once only for continuation/remediation
VALIDATE → use fresh context for initial independent validation; reuse the same worker for
targeted revalidation of its own findings; start fresh only when materially changed
implementation requires a new independent full validation
REVIEW → never reuse; start fresh after remediation
```

After the allowed ARCHITECT or BUILD reuse, dispatch a fresh worker on the same branch and
workstream. Never reuse one role as another.

## Advancement

- BUILD requires approval, the active `solo/<slug>` branch, and its exact task artifact.
- VALIDATE starts only after every BUILD task is `DONE`.
- REVIEW starts only after `VALIDATION.md` is `PASS`.
- `READY_FOR_USER` requires all tasks `DONE`, validation `PASS`, review `PASS`, zero
  unresolved `CRITICAL`/`IMPORTANT`, and clean/understood Git state.
- A missing or non-canonical artifact blocks advancement.

## Validation remediation

Classify each VALIDATE blocker as `PRODUCT` (production behavior, contract, persistence,
financial semantics, safety, or architecture), `REGRESSION` (supported behavior or tests),
or `TOOLING` (typing, lint, formatting, fixtures, harness, selectors, or validation
infrastructure only).

After remediation, rerun checks by classification:

- `PRODUCT` → affected validation plus broader checks reasonably invalidated.
- `REGRESSION` → affected tests/checks plus directly dependent validation.
- `TOOLING` → only the failed check and directly affected tests, unless production code changed.

Previously passing independent evidence remains valid unless remediation could reasonably
invalidate it. Do not rerun the entire validation matrix automatically.

Validation and review findings normally return to the owning BUILD task; reopen that task
and append remediation evidence to its artifact. Create a new `T###` task only for genuinely
new approved scope that cannot reasonably belong to an existing task.

After REVIEW remediation, rerun only the validation evidence reasonably invalidated by the change, then dispatch a fresh REVIEW worker. Previously passing independent validation remains valid unless the remediation could reasonably invalidate it.

The initial independent VALIDATE pass does not count as a remediation cycle. After two
`BUILD → targeted VALIDATE` remediation cycles in the same workstream, stop automatic
cycling. Classify each remaining issue as `PRODUCT BLOCKER`, `VALIDATION / TOOLING DEBT`,
or `NEW SCOPE`, and report the classification and smallest next action to the developer
before continuing.

## Browser validation

For browser-visible changes, BUILD may self-check and VALIDATE independently verifies:

```text
discover → snapshot/read → interact → verify
```

Use Local Host when connected, or another local browser tool. Repeat affected browser checks
after fixes. Prefer
structured text, snapshots, and request evidence; use diagnostics when relevant and
screenshots only for visual diagnosis. Without a browser-capable tool, record the limitation.

## GIT END

After validation and review pass, report `READY_FOR_USER` and stop for explicit merge
approval. After approval, verify branch/status/diff, commit the feature branch, verify the
commit, switch to `main`, verify `main`, merge `solo/<slug>`, verify the merge, and clean
up only when safe and approved. Do not silently push, force, squash, rebase, or delete
unmerged work. Then append `dispatch/COMPLETED.md` and clear `dispatch/ACTIVE.md`.

If abandoned, preserve/report uncommitted work, obtain approval before destructive cleanup,
return to `main`, and clear `ACTIVE.md`.

## Specialist skills

Load only relevant global skills and name them in the worker header: `tdd` for
strategy/engine correctness or regressions; `improve-codebase-architecture` for advisory
architecture; `vercel-react-best-practices` for React/Next.js; `frontend-design` and
`shadcn` for visual UI; `web-design-guidelines` plus Local Host for UI review; `fastapi`
for FastAPI; and `supabase-postgres-best-practices` for PostgreSQL. Project context,
approved contracts, domain semantics, and Strategy specifications override generic advice.
