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
      "model": "openai/gpt-5.6-luna",
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

## Worker roles

Every dispatch identifies `WORKTREE` as `NONE` and provides the active repository root,
branch/base SHA, workstream paths, owned artifact, dependencies, and role contract.
`WORKTREE` is not a normal role in branch-first SoloFlow.

## Specialist skills and MCPs

Specialist skills are global and opt-in. CodeGraph and Local Host are optional MCPs kept
separate from workflow configuration. Use them when relevant; do not preload unrelated
skills or claim browser validation without a browser-capable tool.

## Legacy station

The old Vike configuration is not part of normal SoloFlow operation. Restore it only by
deliberately changing the active OpenCode configuration.
