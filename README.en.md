# Requirement Orchestrator

**English** · [中文](README.md)

Turn a software request into **independently verifiable** work while a **single controlling agent** stays responsible for scope, dependencies, evidence, and integration. It is an orchestrator, not a prompt generator.

> The specification lives in [`SKILL.md`](SKILL.md); this README is a human-facing guide. On any conflict, `SKILL.md` wins.

## The pain it solves

Multi-step agent work in a real codebase usually fails not from bad logic but from: scope quietly creeping, context lost or stale, silent parallel-write conflicts, "done" declared without evidence, and no single owner of state — worst of all across sessions and platforms (Codex ↔ Claude). This skill pins "decompose → delegate → integrate" into a reproducible, auditable process using **one controlling agent + a ledger + frozen contracts + evidence gates**.

## Requirements & dependencies

- A pure **methodology skill (Markdown only)**: no runtime, no `pip`/`npm` dependencies, nothing to install.
- An **agent host** that can read a skill / `SKILL.md`: Claude Code (omp), Codex, or any agent you point at `SKILL.md`.
- Optional integration: [skills-radar](https://github.com/tomtdhzz/skills-radar) as a "starting capability base" (see `references/knowledge-base.md`).

## Install

```bash
# Option 1: skills CLI (installs into ~/.claude/skills or your agent's skills dir)
npx skills add tomtdhzz/requirement-orchestrator -g -y

# Option 2: clone and symlink into your agent's skills directory
git clone https://github.com/tomtdhzz/requirement-orchestrator.git
ln -s "$PWD/requirement-orchestrator" ~/.claude/skills/requirement-orchestrator
```
Or just have your agent read the repo's `SKILL.md` directly.

## Usage

- **Trigger**: tell the agent "use requirement-orchestrator to analyze/orchestrate this request…"; it also engages automatically when a request needs decomposition + delegation + verification.
- **Pick a mode**: `analyze` (default, read-only blueprint) / `diagnose` / `execute` (needs your explicit authorization to change code) / `challenge`.
- **Process**: the agent reads `SKILL.md`, loads `references/*` on demand, keeps a phased TODO and a ledger, and gates completion on evidence after `execute`.
- **Sedimentation (optional)**: at task end, project lessons are appended automatically to `.ai-work/lessons.md`; whether to **promote general lessons back into this skill** is an opt-in choice (default off, needs your OK, committed separately) — see `references/experience.md`.
- Full walkthroughs: `references/examples/feature-example.md`, `bug-example.md`.

## When to use it

When a requirement, feature, bug, service change, or domain change must be decomposed, delegated to subagents, and controlled through verification and integration. Typical triggers:

- a request holds several candidate tasks, or crosses service/domain boundaries;
- scheduling decisions are needed (order, what can run in parallel);
- a build/deployment failure where repository state may be causal;
- collaboration or control handoff across Codex / Claude.

For a single-root-cause task with no delegation, use the control loop directly — no need to spin up the full machinery.

## The four semantic modes

Modes are **semantic**, not a platform's native Plan Mode; they neither require `EnterPlanMode` nor mandatory Explore/Plan agents.

| Mode | What it does | Edits code? |
|---|---|---|
| `analyze` | Investigate request + code, keep the ledger, produce an execution blueprint | No (default) |
| `diagnose` | Reproduce and explain a failure; separate confirmed root cause / evidence / unknowns / repair paths | No |
| `execute` | Dispatch bounded work, review returned results, complete integration | Yes (**needs separate user authorization**) |
| `challenge` | Test an existing requirement/decomposition/design for omissions and risks, without changing its confirmed goal | No |

`analyze` and `diagnose` are read-only **with respect to product code and existing artifacts** — they still write their own deliverables (`docs/prd/`, `docs/tech-design/`) and `.ai-work/` state. Neither a platform permission or native-plan approval, nor the implementation intent of the triggering request ("add login for me"), authorizes editing anything else; entering `execute` requires a separate user instruction.

## Control loop

1. Establish goal, scope, acceptance, constraints, and verified code facts; investigate what you can, ask the user only for decisions that materially affect scope/acceptance/direction.
2. For a bug/failure, attempt a proportionate local reproduction before asserting a root cause; until reproduced, root-cause claims are hypotheses with unverified items flagged.
3. Choose **one** primary decomposition axis, cut bounded tasks, record dependencies and cross-task contracts explicitly; mirror task groups into a **phased TODO**.
4. Keep exactly one controlling agent: it alone owns the ledger, dispatch, review status, replanning, and final integration.
5. Dispatch only tasks with a complete contract; before writing to anything beyond the local tree, run the **target-system preflight**.
6. Results enter `review` first; verify evidence and boundaries before marking `completed`, or return with specific findings. For bulk/external mutations, verify by **independent read-back + structural delta**, not the tool's reported count.
7. Recompute affected dependencies after discoveries, failures, or scope changes; rework the smallest affected branch.
8. Declare done only when task-level checks, cross-task contracts, and every acceptance scenario pass together.

## Progress surface

Keep a visible **phased TODO** in sync with the ledger so the user sees phase- and step-level progress, not just prose. The ledger is the detailed source of truth (dependencies/scopes/contracts/evidence); the phased TODO is its progress view.

- One phase per decomposition group, plus a final acceptance phase. Each item is one bounded task or verification step, phrased as a 5–10 word outcome.
- Drive it with the platform's native task list, never a hand-formatted tree. Advance from real progress: in-progress on dispatch, done only when the ledger records `completed`.

## Target-system preflight

Before bulk-writing to any system **outside** the local working tree (wiki, ticketing, database, remote API), establish up front:

- write permission and the exact scope the operations need (including delete/move if planned);
- rate limits and whether failures are silent; choose safe concurrency and retry/backoff;
- that the result can be read back to verify.

Discover a missing capability here, not mid-batch. If a required permission/scope is unavailable, treat it as a blocker: state what is missing and the smallest grant that unblocks, and stop before any partial write.

## Non-negotiable boundaries

These are MUST NOT-level invariants; kept few and sharp. Elsewhere, rule force is graded with RFC 2119 (MUST / SHOULD / MAY, meaningful only in capitals) with restraint, and each prohibition is paired with a positive "do instead" — negative-only rules are unreliable for agents.

- Do not treat task-tree position as execution dependency.
- Do not let workers expand their write scope silently.
- Do not run writing agents in parallel unless dependencies are resolved, write scopes don't overlap, shared contracts are frozen, and validation runs independently.
- A worker may submit for review; only the controller marks `completed`.
- When changing existing artifacts, don't clear-and-rewrite anything the task didn't author and can't regenerate — insert/patch in place with a rerun-safe marker, verified by read-back.
- Don't assert a derived value beyond its verified source; mark unverified derivations rather than fabricating.
- Don't activate Trellis merely because `.trellis/` exists.
- Don't hand off a deliverable without consuming it as its reader will (render docs; run README/usage commands as written).
- Don't call a standalone project done without a working README, LICENSE, and `.gitignore`, or with internal `.ai-work/` committed.

## Layout

```
requirement-orchestrator/
├── SKILL.md                      # the spec: modes, control loop, progress surface, preflight, boundaries (authoritative)
├── README.md / README.en.md      # human guide (中文 / English)
├── agents/openai.yaml            # display name + default prompt
├── docs/experience-loop.md       # design rationale for the experience loop
└── references/                   # loaded on demand, not read all at once
    ├── spec-driven.md            # spec model: requirements / acceptance scenarios / contracts; frozen-region delimiter
    ├── tech-design.md            # technical design (how): architecture, alternatives, cross-cutting; TDD/RFC structure
    ├── deliverables.md           # artifact layout: publishable docs vs internal .ai-work; publishable-project scaffold
    ├── context-grounding.md      # ground architecture/deps/conventions/blast-radius as facts before decomposing
    ├── decomposition.md          # axis, bug triage, dependencies, parallel gate, replanning
    ├── verification.md           # evidence standards + controller review + finding routing + no pre-judged review dispatch + re-review convergence and round cap + anti-fabrication
    ├── challenge.md              # stress-testing: quality dimensions + omission/risk list
    ├── ledger.md                 # requirement ledger: machine-state YAML, state machine, provenance, handoff snapshot
    ├── agent-contract.md         # subagent contract: required dispatch fields, worker duties, controller review (short form)
    ├── mutation.md               # safe mutation: non-destructive, idempotent, pilot→batch, read-back verify (single-line edits included)
    ├── knowledge-base.md         # consult a capability base first (optional, e.g. skills-radar)
    ├── experience.md             # experience loop: auto project lessons; opt-in skill self-improvement (off by default)
    ├── examples/                 # end-to-end: feature-example.md, bug-example.md
    ├── codex-adapter.md          # Codex platform adapter
    ├── claude-adapter.md         # Claude platform adapter
    └── trellis-adapter.md        # Trellis adapter (optional)
```

## Core concept: the ledger

A **shared source of truth** across agents / sessions / platforms. It stores decisions and evidence, not hidden reasoning or full chat transcripts.

- A small single-session task may keep it in the conversation; multi-agent / cross-session / cross-platform work persists it to disk (`.ai-work/ledger.md`, gitignored for a publishable project).
- Readable Markdown wrapping one controlled YAML block: `request` / `analysis` / `tasks` / `integration`.
- State machine: `pending → in_progress → review → completed`, plus `blocked` / `failed`; workers report only `review`/`blocked`/`failed`, and only the controller records `completed` after verification.
- Before switching controlling platform, write and confirm a **handoff snapshot**; the old controller stops dispatching only after the new one acknowledges it.

## Limitations & non-goals

- **No enforcement.** This is a pure prompt artifact: its rules give you an **observable check, not a block**. They let a violation be caught afterwards; they cannot stop one from happening. Real enforcement lives in tooling (an editor that echoes the diff, a pre-commit hook, a CI gate) — this skill neither provides that nor pretends to.
- Consequently [`references/verification.md`](references/verification.md)'s evidence standards are the only part that converts good intentions into something checkable. Remove them and the rest is wording.
- **Resident cost is real.** `SKILL.md` enters context every session, and rule growth crowds out the task's own context. That is why every rule must state a retirement condition (see `Retiring a rule` in [CONTRIBUTING.md](CONTRIBUTING.md)).
- **The parallel gate is a criterion, not a lock.** It tells you whether two writing tasks may run at once; it does not isolate them. Working-tree isolation remains your git habit.
- **The ledger is a file, not a database.** Cross-session continuation depends on it, but nothing arbitrates when it is stale or corrupted — on recovery, treat every `in_progress` task as unverified and re-check its artifacts.
- **Non-goals**: no task store, board, or dashboard; no hosted state (state always lands in the target project's local files); no spec generation of its own (compose it with spec-kit / OpenSpec / BMAD — they produce the spec, this skill governs delegation and acceptance); no strongly consistent cross-platform state (handoff is a snapshot, not a server).

## License

[MIT](LICENSE)

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Changes are graded with RFC 2119, and every rule must state its gap, consequence class, cost location, and **retirement condition** — favoring fewer, sharper rules over an ever-growing checklist.
