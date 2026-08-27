# SoloFlow

SoloFlow is a single-model engineering workflow for AI coding agents.

It provides a long-lived engineering lead, fresh role-based worker contexts, selective
CodeGraph investigation, linked-worktree safety, task receipts, independent validation,
review, and durable project memory without a heterogeneous specialist-agent fleet.

## Quick start

1. Add `/path/to/soloflow/skills` to OpenCode `skills.paths`.
2. Configure `solo-flow` as the default agent and `solo-flow-worker` as its only child.
3. Preserve CodeGraph and Local Host MCPs when available.
4. Run `/init` in a project when project context is missing.
5. Ask SoloFlow to make a change.

Read the [setup guide](docs/OPENCODE.md) before editing configuration.

## Workflow

```text
Small:    SoloFlow → implement → targeted check → done
Feature:  SoloFlow → PLAN → tasks → worktree → BUILD → VALIDATE → REVIEW
Critical: SoloFlow → PLAN → ARCHITECT → approval → tasks → BUILD → VALIDATE → REVIEW
```

SoloFlow explores the repository itself. It uses CodeGraph first when available and
targeted search as a fallback. Local Host is the preferred local browser validator, but
another local browser tool may be used when Local Host is unavailable.

## Documentation

- [Workflow](docs/WORKFLOW.md)
- [Dispatch](docs/DISPATCH.md)
- [Worktrees](docs/WORKTREES.md)
- [OpenCode](docs/OPENCODE.md)

## Repository

This is a standalone local SoloFlow project. It has no GitHub remote by default.
