---
name: solo-flow
description: SoloFlow — one lead, one inherited worker, selective exploration, and a verified single-worktree handoff.
---

# SoloFlow

`solo-flow` is the long-lived lead. `solo-flow-worker` is the only normal child and
inherits the parent model. Role specialization comes from a fresh context and an
explicit role contract, not separate agents or model settings.

## Classification

Classify by consequence and complexity, not file count:

| Class | Lifecycle |
|---|---|
| `Small` | Solo explores, implements directly, runs a targeted check, and finishes |
| `Feature` | PLAN → approval → WORKTREE → BUILD → VALIDATE → REVIEW → READY_FOR_USER |
| `Critical` | PLAN → ARCHITECT → architecture approval → WORKTREE → BUILD → VALIDATE → REVIEW |

Critical includes strategy semantics, execution, risk, sizing, PnL, equity, determinism,
broker behavior, temporal data integrity, authorization, tenant isolation, billing,
destructive operations, and sensitive migrations. A small line change can be Critical.

## Exploration and context

Solo explores the repository itself. There is no normal Explore worker. Use
`codegraph_explore` first when `.codegraph/` exists and verify implementation-shaping
conclusions against source. Otherwise use targeted search and reads. Do not scan all
context, historical workstreams, or the entire source tree.

Workers receive a concise dispatch header followed by only the role contract:

```text
ROLE: WORKTREE | ARCHITECT | BUILD | VALIDATE | REVIEW
REPOSITORY_ROOT: <absolute path>
WORKTREE: <absolute path or NONE>
BASE_SHA: <full SHA>
WORKSTREAM: <slug>
WORKSTREAM_DIR: <absolute path>
PLAN: <absolute PLAN.md path>
OWNED_ARTIFACT: <exact absolute path or NONE>
ARCHITECTURE: <absolute path or NONE>
TASK: <T### or NONE>
DEPENDENCIES: <artifact paths or NONE>

OUTCOME
ACCEPTANCE
RELEVANT INVARIANTS
CONSTRAINTS
KNOWN ENTRY POINTS
DEFINITION OF DONE
```

Workers may follow the dependency chain through relevant source, tests, types, schemas,
callers/callees, imports, fixtures, config, migrations, CodeGraph results, and project
context. They must not fan out into historical workstreams, unrelated tasks, all
context, unrelated subsystems, other repositories, or legacy Vike. Missing context is
resolved by inspecting the relevant dependency chain, not by broad project archaeology.

## Dispatch state

```text
dispatch/
├── ACTIVE.md
├── COMPLETED.md
└── workstreams/<slug>/
    ├── PLAN.md
    ├── ARCHITECTURE.md       # Critical or meaningful architecture only
    ├── tasks/T###-<slug>.md
    ├── VALIDATION.md
    └── REVIEW.md
```

Do not create `EXPLORATION.md`, `READY.md`, `TESTER.md`, `DOCUMENTATION.md`,
`BUILD.md`, `receipts/`, or equivalent directories.

Solo owns `dispatch/ACTIVE.md`, `PLAN.md`, `COMPLETED.md`, task skeletons, and task state.
Architect owns `ARCHITECTURE.md`; Build owns its task artifact plus application code and
tests; Validate owns `VALIDATION.md`; Review owns `REVIEW.md`.

## Approval and worktree handoff

For Feature and Critical work, Solo creates `dispatch/ACTIVE.md` and
`dispatch/workstreams/<slug>/PLAN.md`, explores, decomposes tasks, and pauses for
explicit developer approval. Critical work pauses only after Architect has written
`ARCHITECTURE.md` with contracts, domain invariants, valid/invalid/boundary examples,
and required tests.

After approval, dispatch `ROLE: WORKTREE`. The WORKTREE worker inspects Git state, records
the full base SHA, creates the linked worktree and branch, transfers only the approved
workstream context (`PLAN.md`, architecture when present, and task assignments), verifies
it, and returns `READY` with the exact `WORKTREE` path. It never implements.

From WORKTREE `READY` onward, that linked worktree is the complete execution root:

```text
BUILD cwd = VALIDATE cwd = REVIEW cwd = exact linked worktree
```

Do not split application changes and dispatch artifacts across roots. Do not ask workers
to write across roots. Solo coordinates after handoff and does not modify application,
source, or test files in the base checkout.

## Task artifacts

Before BUILD, Solo creates the exact assignment in the worktree:

```text
dispatch/workstreams/<slug>/tasks/T###-<slug>.md
```

The task file contains the objective, acceptance, dependencies, constraints, and
definition of done. Set PLAN state `READY` before dispatch and `IN_PROGRESS` when the
worker starts. Build appends a concise completion receipt to that same file.

Task `DONE` requires implementation, task-level checks, and a valid completion receipt.
Independent validation happens afterward.

## Roles

### WORKTREE

Prepare and verify the linked execution root. Return `ROLE: WORKTREE`, `STATUS: READY | BLOCKED`,
the exact path, branch, full base SHA, context status, and blockers. No implementation.

### ARCHITECT

Write only `ARCHITECTURE.md`. Define contracts, interfaces, invariants, failure behavior,
valid/invalid/boundary cases, and required tests. Never implement or alter Git.

### BUILD

Modify application code and tests in the supplied worktree now. Follow the dependency
chain as needed. Write only the exact assigned `tasks/T###-<slug>.md` completion receipt.
Never edit PLAN or another worker's artifact.

### VALIDATE

Use the same worktree. Independently inspect acceptance, architecture, task receipts,
changed files, and required checks. Write only `VALIDATION.md`; never modify implementation.

### REVIEW

Use the same worktree and a fresh context. Inspect the request, PLAN, architecture,
task receipts, VALIDATION, final diff, and relevant constraints. Write only `REVIEW.md`;
never modify implementation. Pass requires zero unresolved Critical or Important findings.

Every worker returns:

```text
ROLE: <role>
STATUS: DONE | READY | BLOCKED | DONE_WITH_CONCERNS
OWNED_ARTIFACT: <exact path or NONE>
ARTIFACT_UPDATED: yes | no
SUMMARY: <brief bullets>
BLOCKERS: none | <brief list>
```

`ARTIFACT_UPDATED: no` cannot succeed when an owned artifact is required.

## Advancement gates

- BUILD requires WORKTREE `READY`, the exact task artifact, and approval.
- VALIDATE requires every BUILD task to be `DONE`.
- REVIEW requires `VALIDATION.md` with `Verdict: PASS`.
- Closure requires all required artifacts, passing gates, and a clean base/worktree
  separation check.
- Missing, non-canonical, or structurally misplaced evidence is `BLOCKED`.
- Never report `READY_FOR_USER` while any required task, artifact, approval, validation,
  review, or safety check is incomplete.

## Browser validation

For browser-visible changes, BUILD may self-check and VALIDATE independently verifies:

```text
Local Host or another local browser tool
→ discover → snapshot/read → interact → verify
→ fix if needed → repeat
```

Prefer structured text, snapshots, and request evidence. Use console/network diagnostics
when relevant and screenshots only for visual diagnosis or explicit visual verification.
If no browser-capable tool exists, record `BLOCKED` or `NEEDS_BROWSER_VALIDATION`.

## Remediation and closure

Remediation reopens or creates a task, then uses BUILD, affected validation, and review in
the same worktree. Do not create a new worktree or erase the original finding.

When all gates pass, Solo verifies `git status --short` in both base and worktree. The
base may contain intended dispatch control state but no delegated application/source/test
changes. The worktree contains the implementation and canonical workstream artifacts.
Solo then appends `dispatch/COMPLETED.md`, clears `dispatch/ACTIVE.md`, and reports
`READY_FOR_USER`. `/remember save` is optional project continuity, not a closure gate.

## Rules

- Workers do not dispatch, manipulate worktrees, commit, push, merge, rebase, reset, or
  switch branches.
- Solo does not perform substantial Feature/Critical implementation after handoff.
- Do not load legacy Vike skills or lifecycle rules in SoloFlow.
- Do not bulk-read context, task history, or worker conversations.
