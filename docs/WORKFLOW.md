# SoloFlow Workflow

SoloFlow uses one long-lived lead and fresh same-model worker contexts on one active
repository branch.

| Component | Responsibility |
|---|---|
| `solo-flow` | Developer interaction, exploration, planning, Git lifecycle, delegation, and closure |
| `solo-flow-worker` | Fresh `ARCHITECT`, `BUILD`, `VALIDATE`, or `REVIEW` context |
| `PLAN.md` | Workstream control plane and task state |
| `tasks/T###-*.md` | Implementation assignment and completion receipt |
| `VALIDATION.md` | Independent acceptance evidence |
| `REVIEW.md` | Independent review findings |
| `memory.md` | Durable project continuity after closure |

## Classification

### Small

Solo explores, implements directly, runs a targeted check, and finishes. No approval or
feature branch is required unless the change proves more consequential.

### Feature

```text
Solo → PLAN → task decomposition → developer approval
→ git switch -c solo/<slug> → BUILD → VALIDATE → REVIEW → READY_FOR_USER
→ merge approval → GIT END → /remember save
```

### Critical

```text
Solo → PLAN → ARCHITECT → architecture/domain approval
→ git switch -c solo/<slug> → BUILD → VALIDATE → REVIEW → READY_FOR_USER
```

Critical work freezes semantics, invariants, examples, and required tests before coding.

## Context

Solo explores broadly with CodeGraph first when available and targeted source inspection
otherwise. Workers receive the workstream contract and may follow task-relevant
dependencies without unrelated repository archaeology. Tracked project context is
naturally present on the feature branch; no context-copy mechanism is needed.
