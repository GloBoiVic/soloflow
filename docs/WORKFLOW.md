# SoloFlow Workflow

SoloFlow uses one long-lived lead and fresh same-model worker contexts on one active
repository branch.

| Component | Responsibility |
|---|---|
| `solo-flow` | Developer interaction, exploration, planning, Git lifecycle, delegation, and closure |
| `solo-flow-worker` | Fresh `ARCHITECT`, `BUILD`, `VALIDATE`, or `REVIEW` context |
| `PLAN.md` | Workstream control plane and task state |
| `tasks/T###-*.md` | Post-approval implementation assignment and completion receipt |
| `VALIDATION.md` | Original independent acceptance evidence |
| `REVIEW.md` | Original independent review findings |
| `remediations/R###-*/` | Immutable bounded BUILD → VALIDATION → REVIEW chain |

## Classification

### Small

Solo explores, implements directly, runs a targeted check, and finishes. No approval or
feature branch is required unless the change proves more consequential.

### Feature

```text
Solo → PLAN_PENDING_APPROVAL (no tasks) → developer approval
→ git switch -c solo/<slug> → create T001 READY → BUILD → VALIDATE → REVIEW → READY_FOR_USER
→ merge approval → GIT END → close dispatch
```

### Critical

```text
Solo → PLAN → ARCHITECT → DEVELOPER_APPROVAL (no tasks)
→ developer approval → git switch -c solo/<slug> → create T001 READY
→ BUILD → VALIDATE → REVIEW → READY_FOR_USER → merge approval → GIT END
```

Critical work freezes semantics, invariants, examples, and required tests before coding.
Only the planning artifacts exist before approval; PLAN/ARCHITECTURE completion does not
authorize BUILD dispatch.

## Context

Solo explores broadly with CodeGraph first when available and targeted source inspection
otherwise. Workers receive the workstream contract and may follow task-relevant
dependencies without unrelated repository archaeology. Tracked project context is
naturally present on the feature branch; no context-copy mechanism is needed.

## Remediation

Completed execution and evidence artifacts are never reopened or overwritten. An approved-
scope defect creates the next sequential `remediations/R###-<slug>/` directory with
`BUILD.md`, `VALIDATION.md`, and `REVIEW.md`. A new requirement or architecture expansion
returns to the relevant approval gate. BUILD worker/session reuse and the existing retry cap
remain in force; only the artifact is new.
