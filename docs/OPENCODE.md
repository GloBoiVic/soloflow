# SoloFlow in OpenCode

## Active identities

```text
solo-flow (primary) → solo-flow-worker (subagent)
```

`solo-flow-worker` has no model override and inherits the parent model. Configure the
parent once. Do not assume variant propagation until verified against the installed
OpenCode version. The worker cannot recursively spawn workers.

## Minimal configuration

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "default_agent": "solo-flow",
  "subagent_depth": 1,
  "skills": { "paths": ["/path/to/soloflow/skills"] },
  "agent": {
    "solo-flow": {
      "mode": "primary",
      "model": "opencode/gpt-5.6-luna",
      "variant": "medium",
      "permission": { "task": { "*": "deny", "solo-flow-worker": "allow" } }
    },
    "solo-flow-worker": {
      "mode": "subagent",
      "hidden": true,
      "permission": { "*": "allow", "task": "deny" }
    }
  }
}
```

Change the parent model once. Keep MCP configuration separate from workflow identity.
Preserve CodeGraph and Local Host when available.

## Worker dispatch contract

Every dispatch begins with:

```text
ROLE
REPOSITORY_ROOT
WORKTREE
BASE_SHA
WORKSTREAM
WORKSTREAM_DIR
PLAN
OWNED_ARTIFACT
ARCHITECTURE
TASK
DEPENDENCIES
```

Then include only outcome, acceptance, relevant invariants, constraints, known entry
points, and definition of done. Workers may inspect task-relevant dependencies but do
not perform general repository exploration. An unresolved required dependency is
`BLOCKED`.

Canonical artifacts are pre-created by Solo: `ARCHITECTURE.md`,
`tasks/T###-<slug>.md`, `VALIDATION.md`, or `REVIEW.md`. Workers never invent alternate
paths or `receipts/` directories. Solo verifies the exact artifact and terminal status
before advancing state.

## Legacy station

The old Vike configuration is not part of normal SoloFlow operation. Restore it only by
deliberately changing the active OpenCode configuration back to the legacy setup.
