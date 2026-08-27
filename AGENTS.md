# SoloFlow repository instructions

SoloFlow is the active, single-model engineering workflow. Keep this repository small
and focused. Do not import the legacy Vike lifecycle or specialist-agent hierarchy.

- `solo-flow` is the long-lived primary controller.
- `solo-flow-worker` is the only normal child and inherits the parent model.
- Use CodeGraph first for structural investigation when an index is available.
- Use Local Host, or another local browser tool, for browser-visible validation when available.
- Preserve task receipts and branch-first Git safety; never commit, push, merge, reset, or clean up automatically.
- Do not persist secrets, transcripts, or large command output.
