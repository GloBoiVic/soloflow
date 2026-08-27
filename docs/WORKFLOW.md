# SoloFlow Workflow

SoloFlow separates a durable lead context from fresh implementation contexts while
keeping the same model and project intent.

## Components

| Component | Responsibility |
|---|---|
| `solo-flow` | Developer interaction, exploration, planning, delegation, state, and closure |
| `solo-flow-worker` | Fresh `ARCHITECT`, `BUILD`, `VALIDATE`, or `REVIEW` context |
| `PLAN.md` | Workstream control plane and task state |
| `tasks/T###-*.md` | Durable implementation assignment and completion receipt |
| `VALIDATION.md` | Independent acceptance evidence |
| `REVIEW.md` | Independent review findings |
| `memory.md` | Optional durable project continuity |

## Classification

### Small

An obvious, localized, low-consequence change. SoloFlow implements directly and runs a
targeted check. No full workstream is required unless useful.

### Feature

Normal product or engineering work. SoloFlow explores, creates a plan, waits for approval,
creates a linked worktree, decomposes coherent tasks, dispatches Build workers, validates,
reviews, remediates when needed, and ends `READY_FOR_USER`.

### Critical

Work where an incorrect interpretation or failure has material consequences. SoloFlow
dispatches an Architect, obtains developer approval of semantics or architecture, then
uses the Feature flow.

## Context

SoloFlow uses CodeGraph first for structural investigation and verifies important findings
against source. If CodeGraph is unavailable, it uses targeted search. Workers receive
focused queries, explicit non-indexed context paths, relevant dependency receipts, and
bounded write/validation instructions. Workers follow relevant dependencies without
turning the task into repository-wide exploration; an unresolved required dependency is
reported as `BLOCKED`.

## Validation

Browser-visible changes require Local Host or another local browser tool when available:

```text
discover → snapshot/read → interact → verify
```

If no browser-capable tool exists, record `BLOCKED` or `NEEDS_BROWSER_VALIDATION` rather
than claiming UI validation passed.
