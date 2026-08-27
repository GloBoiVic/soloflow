# SoloFlow Dispatch

SoloFlow keeps central state in `PLAN.md` and durable implementation history in task
receipts.

## Lifecycle

```text
request → classification → Solo exploration → PLAN → approval when required
→ linked worktree → BUILD → VALIDATE → REVIEW → remediation → READY_FOR_USER
```

Small one-liners may skip dispatch. Every other code-changing request creates or resumes
`dispatch/ACTIVE.md` and `dispatch/workstreams/<slug>/PLAN.md` before implementation.
Never create these files at the repository root.

## Artifacts

```text
dispatch/
├── ACTIVE.md
├── COMPLETED.md
└── workstreams/<slug>/
    ├── PLAN.md
    ├── ARCHITECTURE.md       # optional
    ├── tasks/T###-<slug>.md
    ├── VALIDATION.md
    └── REVIEW.md
```

There is no normal Explore worker, `EXPLORATION.md`, `READY.md`, Tester worker, or
Documenter worker. `solo-flow-worker` handles the WORKTREE role when an execution root
is required.

`ACTIVE.md` is a small root pointer. `PLAN.md` owns scope, acceptance, task state,
current phase, next action, and worktree details. Each worker writes only its assigned
artifact. Later workers receive only relevant dependency receipts.

After approval, WORKTREE prepares and verifies the linked execution root. BUILD, VALIDATE,
and REVIEW all use that same exact cwd. Before BUILD, Solo creates the exact assignment
at `dispatch/workstreams/<slug>/tasks/T###-<slug>.md` in that worktree. BUILD must not
start without it and must not invent a root-level receipt or `receipts/` directory.
VALIDATE starts only after every BUILD task is `DONE`; REVIEW starts only after validation
is `PASS`.

Solo performs closure checks from `REPOSITORY_ROOT`: every required artifact exists,
`dispatch/COMPLETED.md` was appended, and `dispatch/ACTIVE.md` is cleared. Missing
evidence keeps the workstream active and cannot produce `READY_FOR_USER`.

## Receipts

Each receipt has one canonical owner. Build receipts belong to task files; cross-task
validation belongs to `VALIDATION.md`; review findings belong to `REVIEW.md`. Other
artifacts reference receipts rather than copying them.

## Closure

Solo appends `COMPLETED.md`, clears `ACTIVE.md`, and leaves the branch/worktree available
for developer inspection. `/remember save` is optional and is not a closure gate.
