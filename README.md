# SoloFlow

SoloFlow is a single-model engineering workflow for AI coding agents.

It provides a long-lived engineering lead, fresh role-based worker contexts, selective
CodeGraph investigation, branch-first execution, task receipts, independent validation,
review, and durable project memory without a heterogeneous specialist-agent fleet.

## Get started

Install the skills:

```bash
npx skills add GloBoiVic/soloflow
```

Then configure OpenCode with `solo-flow` as the default agent and
`solo-flow-worker` as its only child. See the [OpenCode setup guide](docs/OPENCODE.md)
for the minimal configuration.

Run `/init` in a project when project context is missing, then ask SoloFlow to make a
change.

Read the [setup guide](docs/OPENCODE.md) before editing configuration.

## Workflow

```text
Small:    SoloFlow → implement → targeted check → done
Feature:  SoloFlow → PLAN → approval → solo/<slug> → BUILD → VALIDATE → REVIEW
Critical: SoloFlow → PLAN → ARCHITECT → approval → solo/<slug> → BUILD → VALIDATE → REVIEW
```

SoloFlow explores the repository itself. It uses CodeGraph first when available and
targeted search as a fallback. Local Host is the preferred local browser validator, but
another local browser tool may be used when Local Host is unavailable.

## Specialist skills

SoloFlow loads these optional global skills only when a task needs them:

| Work | Skill |
|---|---|
| Strategy/engine correctness | `tdd` |
| Architecture | `improve-codebase-architecture` |
| React/Next.js | `vercel-react-best-practices` |
| Visual UI | `frontend-design`, `shadcn` |
| UI review | `web-design-guidelines` + Local Host |
| FastAPI | `fastapi` |
| PostgreSQL | `supabase-postgres-best-practices` |

Specialist advice is advisory. Developer-approved behavior, project architecture,
workstream contracts, and Strategy specifications take precedence.

## Documentation

- [Workflow](docs/WORKFLOW.md)
- [Dispatch](docs/DISPATCH.md)
- [Worktrees](docs/WORKTREES.md)
- [OpenCode](docs/OPENCODE.md)

## Optional integrations

SoloFlow works without external integrations. When available, CodeGraph improves
structural investigation and Local Host provides local browser validation. Other local
browser tools can be used when Local Host is unavailable.

## License

SoloFlow is available under the [MIT License](LICENSE).
