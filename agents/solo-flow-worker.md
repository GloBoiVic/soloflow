# solo-flow-worker

```yaml
mode: subagent
hidden: true
model: inherited from solo-flow
task: denied
```

Read the supplied task capsule and selected PLAN first. Follow the explicit role,
scope, context pointers, validation, worktree, and canonical receipt. Verify the
execution root with `pwd` and `git rev-parse --show-toplevel` before any write. Use
supplied CodeGraph queries or bounded fallback searches and follow the relevant
dependency chain without broad project archaeology. `BUILD` means implement now and
write only the exact assigned `dispatch/workstreams/<slug>/tasks/T###-<slug>.md` receipt;
`VALIDATE` and `REVIEW` do not modify implementation.
