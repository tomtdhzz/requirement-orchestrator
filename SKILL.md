---
name: requirement-orchestrator
description: Use when a software requirement, feature, bug, service change, or domain change must be decomposed into bounded work, delegated to subagents, and controlled through verification and integration across Codex or Claude.
---

# Requirement Orchestrator

Turn a software request into independently verifiable work while keeping one controlling agent responsible for scope, dependencies, evidence, and integration. This skill is an orchestrator, not a prompt generator.

## Select a semantic mode

- `analyze`: investigate the request and codebase, maintain the ledger, and produce an execution blueprint without modifying code. This is the default when implementation was not requested.
- `diagnose`: reproduce and explain a reported failure, separating confirmed root cause, evidence, unknowns, and recommended repair paths. Diagnosis is the deliverable; do not modify code.
- `execute`: dispatch bounded work, review returned results, and complete integration. Enter only when the user authorized implementation or execution.
- `challenge`: test an existing requirement, decomposition, or design for omissions and risks without changing its confirmed product goal. See [references/challenge.md](references/challenge.md).

These are skill-level semantic modes. They do not invoke or require a platform's native Plan Mode, `EnterPlanMode`, or mandatory Explore/Plan agents. Use native modes only when independently useful and compatible with the user's constraints.

For a single-root-cause task with no delegation, use the control loop directly. Read [references/spec-driven.md](references/spec-driven.md) to shape the specification (requirements, acceptance scenarios, contracts). Read [references/context-grounding.md](references/context-grounding.md) before decomposing in an existing codebase. Read [references/decomposition.md](references/decomposition.md) when there is more than one candidate task, a service/domain boundary, a scheduling decision, or a build/deployment failure where repository state may be causal. Read [references/verification.md](references/verification.md) for acceptance-evidence standards. Read [references/ledger.md](references/ledger.md) when work has multiple tasks, agents, sessions, or platforms. Read [references/mutation.md](references/mutation.md) when execution changes an existing artifact — a rewrite, a delete, a change fanned across several artifacts, or a write to an external system; its read-back rule covers even a single-line edit. Worked runs: [references/examples/feature-example.md](references/examples/feature-example.md) and [references/examples/bug-example.md](references/examples/bug-example.md).

In `diagnose`, report the observed or reproduced failure, confirmed root cause and evidence, unverified items, impact, and recommended repair paths. Stop there unless the user separately authorizes `execute`.

## Control loop

1. Establish the goal, scope, acceptance criteria, constraints, and verified code facts. Ground unknowns first ([references/context-grounding.md](references/context-grounding.md)), then express the result as a specification — requirements, acceptance scenarios, and contracts ([references/spec-driven.md](references/spec-driven.md)). Investigate discoverable facts; ask the user only for decisions that materially affect scope, acceptance, or direction. Before decomposing, prefer reusing an existing capability over researching from scratch: search the repo and the installed skills for prior art, and — only if a capability knowledge base is already configured — query it for the relevant capabilities and a same-domain comparison ([references/knowledge-base.md](references/knowledge-base.md)). Record the capabilities you chose, with where each came from, in the ledger; never record a capability you did not actually find.
   - Read prior project lessons if present ([references/experience.md](references/experience.md)) and treat them as candidate facts to verify, not gospel.
   - For a non-trivial build, after the spec produce a **technical design** ([references/tech-design.md](references/tech-design.md)) — architecture, detailed design, alternatives, cross-cutting concerns — and persist artifacts under the `docs/` layout ([references/deliverables.md](references/deliverables.md)).
2. For a reported bug or failure, attempt a proportionate local reproduction before claiming a root cause. Compare the observed failure with the report. Until reproduced or supported by equivalent direct evidence, label root-cause statements as hypotheses and state what remains unverified.
3. Choose one primary decomposition axis and create bounded tasks. Derive the shared contracts and freeze them before dependent or parallel work ([references/spec-driven.md](references/spec-driven.md)). Record dependencies and cross-task contracts explicitly. Mirror the resulting task groups into a phased TODO (see Progress surface).
4. Keep exactly one controlling agent. The controller owns the ledger, dispatch decisions, review status, replanning, and final integration.
5. Dispatch only tasks with a complete contract. Read [references/agent-contract.md](references/agent-contract.md) before the first dispatch. Before dispatching writes to any system beyond the local working tree, run the target-system preflight below.
6. Accept a worker result into `review`, verify its evidence and boundaries against the acceptance-evidence standards ([references/verification.md](references/verification.md)), then mark it `completed` or return it with specific findings. For a bulk or external mutation, verify by independent read-back and a structural delta, not the tool's reported success count.
7. Recompute affected dependencies after discoveries, failures, or scope changes. Rework the smallest affected branch.
8. Declare the request complete only after task-level checks, cross-task contracts, and every acceptance scenario pass together; the end-to-end gate is defined in [references/verification.md](references/verification.md). On completion, append any qualifying project-specific lessons to the experience log ([references/experience.md](references/experience.md)). Improving the skill itself from the run is a separate **opt-in** action — offered only when a lesson is general and recurring, applied only if the user chooses it, never automatic (see [references/experience.md](references/experience.md)).

## Progress surface

Keep a visible phased TODO in sync with the ledger so the user sees phases and step-level progress, not only prose. The ledger stays the detailed source of truth (dependencies, scopes, contracts, evidence); the phased TODO is its progress view.

- Initialize one phase per decomposition group, then a final acceptance phase (for a bug flow, `对照` / `修复` / `验证`; for a feature flow, the feature groups followed by `全量验收`). Each item is one bounded task or verification step, phrased as an outcome in 5–10 words.
- Drive it with the platform's native task list, never a hand-formatted tree. On Claude/omp use the `todo` tool: `init` with `list: [{phase, items}]`, then `start`/`done` per item. On Codex use its plan/update-plan mechanism.
- Advance state from real progress: mark an item in progress when its task is dispatched, and done only after the controller records `completed` in the ledger. Phase counts (`N/M`) then track verified acceptance, not dispatch.
- Keep the two consistent: one TODO item corresponds to exactly one ledger task or verification step. Recompute both together during replanning.
- In `analyze` and `diagnose`, the phases are investigation or diagnosis steps and completing them modifies no code.

## Deliverable artifacts

Separate publishable docs from internal orchestration state: reader-facing docs go under `docs/` (`docs/prd/` spec, `docs/tech-design/` design); the ledger, plan, and lessons are internal working state under `.ai-work/` — never under `docs/`, and gitignored for a publishable project. See [references/deliverables.md](references/deliverables.md).

For any non-trivial build, produce a technical design after the spec and before `execute`: architecture, detailed design, alternatives considered, and cross-cutting concerns ([references/tech-design.md](references/tech-design.md)). Skip it for a single-root-cause fix where the spec plus a task list suffices.

Self-review every generated Markdown deliverable before handoff: re-read it and confirm it renders as the reader will see it — structure intact, fenced/Mermaid blocks actually parse (render them, don't eyeball), links resolve. A parse/render failure is a defect to fix now. See [references/deliverables.md](references/deliverables.md) and the `Document / diagram artifact` row in [references/verification.md](references/verification.md).

When the deliverable is a standalone project, "done" includes being publishable: a README (what/why, prerequisites, install, usage examples, config, architecture, test, limitations, license, disclaimer), a LICENSE, and a `.gitignore` that excludes build output, secrets, and `.ai-work/`. Verify the README's commands actually run. See [references/deliverables.md](references/deliverables.md).

## Target-system preflight

Before dispatching writes to any system other than the local working tree — a wiki, ticketing system, database, or remote API — verify up front:

- write permission and the exact scope the planned operations need, including delete or move if the plan requires them;
- rate limits and whether failures are silent; choose safe concurrency and a retry or backoff;
- that the result can be read back to verify.

Discover a missing capability here, not mid-batch. If a required permission or scope is unavailable, treat it as a blocker: state exactly what is missing and the smallest grant that unblocks, and stop before any partial application.

## Platform routing

- For Codex, read [references/codex-adapter.md](references/codex-adapter.md).
- For Claude, read [references/claude-adapter.md](references/claude-adapter.md).
- When transferring control between them, write and confirm a handoff snapshot before the old controller stops dispatching.
- Trellis is optional. Read [references/trellis-adapter.md](references/trellis-adapter.md) only when its escalation signals apply or the user requests it.

## Non-negotiable boundaries

These are MUST NOT-level invariants regardless of phrasing: few, sharp, and each paired with the positive default to use instead. Elsewhere, force is graded with RFC 2119 keywords, meaningful only in capitals. How to author a rule — grading, pairing, admission and retirement — is in `CONTRIBUTING.md`, not here.

- Do not treat task-tree position as execution dependency. (Do instead: order work only by a recorded `depends_on`.)
- Workers write only inside their `write_scope`; an out-of-scope need is reported to the controller, not taken. (Disclosing an out-of-scope edit does not authorize it.)
- Do not run writing agents in parallel unless dependencies are resolved, no two write scopes share a compile/test target, shared contracts are frozen, and validation is independent. (Different files inside one package/module/test target are not independent.)
- A worker may submit work for review; only the controller may mark it completed.
- When changing existing artifacts, do not clear-and-rewrite anything the task did not author and cannot regenerate; insert or patch in place, mark it with a rerun-safe marker, and verify by independent read-back.
- Do not assert a derived value — a classification, label, summary, risk rating, or recommendation — beyond its verified source; mark an unverified derivation as such rather than fabricating a value.
- Do not activate Trellis merely because `.trellis/` exists. (Do instead: adopt it only when its escalation signals apply or the user asks.)
- `analyze` and `diagnose` are read-only **with respect to product code and existing artifacts**; they still write their own deliverables (`docs/prd/`, `docs/tech-design/`) and `.ai-work/` state. Neither a platform permission or native-plan approval, nor the implementation intent of the request that started this work, authorizes editing anything else: "add login for me" authorizes analyzing that request, not editing in the same response. Enter `execute` only after a separate user instruction authorizes implementation.
- Do not hand off a deliverable without consuming it as its reader will — render the doc, and run the README/usage commands from the repo root exactly as written. (Do instead: render diagrams, run the commands, fix breaks before handoff.)
- Do not call a standalone project done without a working README, LICENSE, and `.gitignore`, or with internal `.ai-work/` artifacts committed. (Do instead: ship the publishable scaffold; keep orchestration state out of the repo.)
