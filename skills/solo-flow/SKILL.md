---
name: solo-flow
description: SoloFlow — one lead, one inherited worker, branch-first execution, and risk-based dispatch.
---

# SoloFlow

`solo-flow` is the long-lived lead. `solo-flow-worker` is the only normal child and
inherits the parent model. Roles are `ARCHITECT`, `BUILD`, `VALIDATE`, and `REVIEW`.
Every worker dispatch uses a fresh context; canonical artifacts provide continuity.

## Classify

Classify by consequence and scope, not file count:

| Class      | Lifecycle                                                                                                             |
| ---------- | --------------------------------------------------------------------------------------------------------------------- |
| `Small`    | Solo explores, implements directly, runs a targeted check, and finishes                                               |
| `Feature`  | PLAN → approval → GIT START → create tasks → BUILD → VALIDATE → REVIEW → merge approval → GIT END                    |
| `Critical` | PLAN → ARCHITECT → reconcile PLAN/ARCHITECTURE → approval → GIT START → create tasks → BUILD → VALIDATE → REVIEW → merge approval → GIT END |

Small changes need no plan approval or branch when genuinely obvious and low consequence.

## Explore

Solo explores. There is no normal Explore worker. Use `codegraph_explore` first when
`.codegraph/` exists and verify important conclusions against source; otherwise use
targeted search and reads. Do not bulk-read context, history, or the source tree.

## Workstream

Feature and Critical work use one active workstream and one `solo/<slug>` branch. Before
approval, planning artifacts only are present:

```text
Feature:  dispatch/workstreams/<slug>/PLAN.md
Critical: dispatch/workstreams/<slug>/PLAN.md + ARCHITECTURE.md
```

After explicit approval and GIT START, Solo creates the BUILD task artifacts:

```text
dispatch/
├── ACTIVE.md
├── COMPLETED.md
└── workstreams/<slug>/
    ├── PLAN.md
    ├── ARCHITECTURE.md       # Critical or meaningful architecture only
    ├── tasks/T###-<slug>.md  # after approval and GIT START
    ├── VALIDATION.md         # original validation
    ├── REVIEW.md             # original review
    └── remediations/R###-<slug>/
        ├── BUILD.md
        ├── VALIDATION.md
        └── REVIEW.md
```

`PLAN.md` owns outcome, classification, scope, acceptance, architecture status, branch
and base SHA, task state, phase, next action, and concerns. It is not an implementation
journal. Task files preserve what original BUILD actually did. Completed task, validation,
review, and remediation artifacts are immutable evidence. Architecture and operational state
remain separate from execution evidence.

Only one Feature/Critical workstream and one `solo/<slug>` branch may be active per
repository. Related follow-up or remediation stays on that branch. An unrelated request
blocks until the current workstream is merged or explicitly abandoned.

`ACTIVE.md` is mutable operational state: keep the workstream, stage, current `T###` or
`R###` when applicable, role, and approval state current without adding execution narratives.

## Approval gate

For approval-gated Feature and Critical work, do not create `tasks/`, a `T###` BUILD
assignment, or a `READY` task before explicit developer implementation approval. Set Feature
to `PLAN_PENDING_APPROVAL` and Critical to `DEVELOPER_APPROVAL`. Developer feedback is not
approval: reconcile it into PLAN/ARCHITECTURE and remain at the gate until an explicit signal
such as `approved`, `looks good, proceed`, `go ahead`, `build it`, or `proceed with
implementation` approves the current reconciled artifacts.

After approval, record the approval in planning state, perform GIT START, verify the branch,
and only then create `tasks/T001-<slug>.md` with `Status: READY` and `Role: BUILD`. Small work
may bypass this gate when its classification permits the direct Small lifecycle.

If a pre-approval task already exists, it is not authorization: do not BUILD from it. Restore
the appropriate approval phase, preserve any execution evidence as historical unauthorized
work, and create the next active BUILD assignment only after approval and GIT START. An empty,
unstarted accidental task may be removed rather than preserved as evidence.

## Approval

For Feature, Solo creates `dispatch/ACTIVE.md` and `PLAN.md`, explores, sets
`PLAN_PENDING_APPROVAL`, shows the plan without creating BUILD tasks, and waits for explicit
developer approval. Reconcile feedback into PLAN and stop again if approval is not explicit.

For Critical, Solo creates the initial PLAN, explores, dispatches `ARCHITECT`, and waits
until `ARCHITECTURE.md` freezes contracts, domain invariants, valid/invalid/boundary
examples, and required tests. Solo reconciles PLAN and ARCHITECTURE, sets
`DEVELOPER_APPROVAL`, shows the combined contract without creating BUILD tasks, and waits for
explicit developer approval. Reconcile feedback into planning artifacts and stop again if
approval is not explicit.

Before any dispatch or continued work, reconcile material developer feedback into
the canonical PLAN, ARCHITECTURE, BUILD task, or remediation packet. Never leave material
feedback only in conversation context.

## GIT START

After required approval, inspect:

```bash
git branch --show-current
```

Meaningful unrelated dirty state blocks. Do not silently stash, commit, discard, reset,
rebase, or switch branches. With explicit operation confirmation, run:

```bash
git switch -c solo/<workstream-slug>
```

Verify the branch and status, record approval and branch/base SHA in `PLAN.md` and `ACTIVE.md`,
and only then create `tasks/T001-<slug>.md` with `Status: READY` and `Role: BUILD`. Do not
switch branches again until merge or explicit abandonment. The normal repository root and
branch are the execution source of truth for Solo, workers, the editor, dev server, Git diff,
and Local Host.

## Worker dispatch

Every worker call provides only this concise header; canonical artifacts provide detailed
requirements:

```text
ROLE: ARCHITECT | BUILD | VALIDATE | REVIEW
WORKSTREAM: <slug>
BRANCH: solo/<slug>
CWD: <absolute repository root>
TASK: <T### | R### | NONE>
OWNED_ARTIFACT: <exact repository-relative path>
SPECIALIST_SKILLS: <names or none>
```

Invoke the `task` tool with the required `subagent_type: "solo-flow-worker"` argument on
every worker call. The dispatch header is prompt content and does not replace that tool
argument. Include the concise header in the worker prompt; do not omit `subagent_type`.

Do not pass the parent conversation. PLAN or ARCHITECTURE completion alone never authorizes a
BUILD dispatch. For Feature/Critical, BUILD requires explicit approval, completed GIT START,
and its post-gate task artifact; Critical ARCHITECT may run before approval. Workers may follow the task dependency chain through
relevant source, tests, types, schemas, callers, imports, fixtures, config, migrations,
project context, and CodeGraph. They must not perform unrelated repository archaeology.

## Ownership and tasks

| Actor     | Writes                                                               |
| --------- | -------------------------------------------------------------------- |
| Solo      | `ACTIVE.md`, `PLAN.md`, `COMPLETED.md`, task assignments, task state |
| ARCHITECT | `ARCHITECTURE.md`                                                    |
| BUILD     | assigned `tasks/T###-*.md` or `remediations/R###-*/BUILD.md` plus application/tests |
| VALIDATE  | `VALIDATION.md` or `remediations/R###-*/VALIDATION.md`               |
| REVIEW    | `REVIEW.md` or `remediations/R###-*/REVIEW.md`                       |

Solo pre-creates a role artifact only at its authorized lifecycle stage. Only BUILD receives
a `T###` task or `R###` remediation. Original task assignment and completion remain in the
same task file; remediation assignment and completion remain in `BUILD.md`:

```text
READY → IN_PROGRESS → DONE | BLOCKED | DONE_WITH_CONCERNS
```

Once a task is `DONE`, do not reopen or append to it. A VALIDATE or REVIEW finding gets a
new sequential remediation packet under `remediations/`.

Task `DONE` requires implementation, task-level checks, and a complete receipt. Workers
must not edit another role's artifact or create alternate receipt directories.

## Roles

`ARCHITECT` defines contracts, invariants, failure behavior, examples, and required tests;
it never implements or changes Git.

`BUILD` implements the assigned task or remediation and tests on the active branch, follows
relevant dependencies, and writes its assigned completion receipt. It never changes branches
or Git history, and never edits a completed evidence artifact.

`VALIDATE` independently checks acceptance, implementation, receipts, tests, builds, data
checks, determinism, and Local Host when relevant. It writes its assigned validation artifact
once and diagnoses only; it never edits application, test, fixture, selector, harness,
workflow, or other implementation code, or overwrites completed evidence.

`REVIEW` independently checks the request, PLAN, architecture, task receipts, validation,
diff, and constraints. It writes its assigned review artifact once and diagnoses and judges
only; it never edits application, test, fixture, selector, harness, workflow, or other
implementation code, or overwrites completed evidence. Pass requires zero unresolved
`CRITICAL` or `IMPORTANT` findings.

Every worker returns a concise receipt:

```text
ROLE: <role>
STATUS: DONE | PASS | FAIL | BLOCKED | DONE_WITH_CONCERNS
ARTIFACT: <canonical path>
FILES CHANGED: <paths or none>
CHECKS / EVIDENCE: <brief result>
FINDINGS / CONCERNS: <brief result>
```

Solo verifies material claims against Git and canonical artifacts before advancing.

## Worker sessions

Dispatch a fresh worker context for every `ARCHITECT`, `BUILD`, `VALIDATE`, and `REVIEW`
call. Never reuse a prior worker session. Canonical artifacts, the same branch, and the
workstream provide continuity; roles remain independent.

## Advancement

- BUILD requires approval where required, the active `solo/<slug>` branch, and its exact
  post-gate task or remediation artifact.
- VALIDATE starts only after the current BUILD task or remediation is `DONE`.
- REVIEW starts only after the current validation artifact is `PASS`.
- `READY_FOR_USER` requires all original tasks `DONE`, the latest validation chain `PASS`,
  the latest review chain `PASS`, zero
  unresolved `CRITICAL`/`IMPORTANT`, and clean/understood Git state.
- A missing or non-canonical artifact blocks advancement.

## Immutable remediation chains

Classify each VALIDATE or REVIEW finding as `PRODUCT` (production behavior, contract,
persistence, financial semantics, safety, or architecture), `REGRESSION` (supported
behavior or tests), or `TOOLING` (typing, lint, formatting, fixtures, harness, selectors,
or validation infrastructure only).

Completed `tasks/T###-*.md`, root `VALIDATION.md`/`REVIEW.md`, and remediation artifacts are
immutable. Never change a completed conclusion, delete an old finding, reopen an original
task, or reuse an old validation/review receipt.

Classify each finding as an approved-scope `DEFECT` or `NEW SCOPE`. A defect against an
approved requirement, invariant, acceptance criterion, or architecture contract creates the
next unused sequential `remediations/R###-<slug>/`; never reuse an existing remediation ID.
New scope returns to the appropriate developer or architecture approval gate without automatic
remediation.

Pre-create the new `BUILD.md` with a concise packet containing:

```text
Remediation ID
Status
Origin finding and source artifact
Finding severity
Related original task(s)
Approved requirement or invariant violated
Exact remediation outcome
Affected implementation seams
Explicit out-of-scope items
Regression evidence required
Worker Evidence
```

The packet references the finding that caused it, including the prior remediation artifact
when applicable. Remediations are bounded to the demonstrated defect, required direct
changes, and regression coverage; escalate if the approved architecture must materially
change.

Dispatch a fresh, narrowly scoped BUILD worker with the packet, relevant contracts, and
`TASK: R###` / `OWNED_ARTIFACT: remediations/R###-<slug>/BUILD.md`. The worker may follow
directly affected dependencies needed to make the fix, but must not broaden scope. Production
defects, tests, fixtures, selectors, and harness code that require edits remain BUILD-owned.

Solo may directly repair validation environment state that does not change tracked product
or test code, including resetting/recreating the dedicated test database, running its
migrations/seeds, restarting workstream-owned test servers, resolving workstream-owned
ports/processes, and rerunning commands. Solo must not modify application persistence,
schemas, migrations, or terminate unknown/unowned processes as environment remediation.

Every remediation runs its own immutable chain:

```text
remediations/R###-<slug>/BUILD.md
  → remediations/R###-<slug>/VALIDATION.md
  → remediations/R###-<slug>/REVIEW.md
```

VALIDATE checks the remediation acceptance criteria, regression coverage, directly affected
behavior, and relevant gates, then writes only `remediations/R###-<slug>/VALIDATION.md`.
REVIEW independently checks the originating finding, scope, architecture, remediation diff,
BUILD receipt, and validation evidence, then writes only
`remediations/R###-<slug>/REVIEW.md`. Use targeted checks by default; do not automatically
rerun the full matrix. Earlier evidence remains available and is never overwritten.

For either kind of blocking finding, use:
finding → next sequential R### BUILD → R### VALIDATE → R### REVIEW

The reviewer inspects the finding, relevant contract, remediation diff, affected task or
remediation receipt, and validation evidence rather than rereading the entire workstream by
default. If root validation failed before root review began, the first remediation still gets
both artifacts; its review is the initial broad review as needed.

Escalate to full validation or review only when remediation materially changes broad
authority, including architecture, domain or financial semantics, persistence or schema,
API contract semantics, Strategy/Risk/execution/accounting behavior, or substantial
cross-cutting implementation.

A remediation return means a VALIDATE or REVIEW finding that creates the next R### chain.
The initial independent VALIDATE and REVIEW passes do not count as remediation returns.
The existing two-remediation-return cap remains workstream-wide; do not reset it because a
new artifact was created. After the cap, classify each remaining issue as `PRODUCT BLOCKER`,
`VALIDATION / TOOLING DEBT`, or `NEW SCOPE`, and report the smallest next action for developer
approval before continuing.

Critical and Important findings block closure unless resolved, reclassified with justification,
or explicitly accepted by the developer where permitted. Minor findings may be remediated,
accepted, or deferred; they do not automatically create a remediation.

## Browser validation

For browser-visible changes, BUILD may self-check and VALIDATE independently verifies:

```text
discover → snapshot/read → interact → verify
```

Use Local Host when connected, or another local browser tool. Repeat affected browser checks
after fixes. Prefer
structured text, snapshots, and request evidence; use diagnostics when relevant and
screenshots only for visual diagnosis. Without a browser-capable tool, record the limitation.

## GIT END

After validation and review pass, report `READY_FOR_USER` and stop for explicit merge
approval. After approval, verify branch/status/diff, commit the feature branch, verify the
commit, switch to `main`, verify `main`, merge `solo/<slug>`, verify the merge, and clean
up only when safe and approved. Do not silently push, force, squash, rebase, or delete
unmerged work. Then append `dispatch/COMPLETED.md` and clear `dispatch/ACTIVE.md`.

If abandoned, preserve/report uncommitted work, obtain approval before destructive cleanup,
return to `main`, and clear `ACTIVE.md`.

The final `COMPLETED.md` entry may reference the successful remediation chain, but never
replace its earlier receipts.

## Specialist skills

Load only relevant global skills and name them in the worker header: `tdd` for
strategy/engine correctness or regressions; `improve-codebase-architecture` for advisory
architecture; `vercel-react-best-practices` for React/Next.js; `frontend-design` and
`shadcn` for visual UI; `web-design-guidelines` plus Local Host for UI review; `fastapi`
for FastAPI; and `supabase-postgres-best-practices` for PostgreSQL. Project context,
approved contracts, domain semantics, and Strategy specifications override generic advice.
