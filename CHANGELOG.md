# Changelog

All notable changes to this project are documented here.
The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/);
this project uses date-based entries (no semantic version tags yet).

Entries are reader-facing: what changes for someone using the skill. Internal
wording, cross-reference and formatting fixes live in the commit history, which
is the durable record; see `CONTRIBUTING.md`.

## [Unreleased]

### Changed
- **The test-first gate is now observable**: the red run (failing test name + failure text) is recorded in `tasks[].evidence`, and the completion gate requires that record. Previously a finished repo looked identical whether the test came first or was backfilled, so the gate could not fail.
- **Mode selection arbitrates itself**: a request referring to a failure that already happened routes to `diagnose` without the user naming a mode; `analyze` is the default only when nothing has failed. Both requests previously satisfied both modes, and the two deliver different artifacts.
- **A new rule must be reachable from the control-loop step it governs** — that step links the file, or a file it already loads points at it. Filing a rule in the topically right reference is not enough when references load on demand.
- **Never create a second spec source**: when the repo already holds the spec (a framework's directory, a wiki page, an issue), work from it and record its location in the ledger. This skill owns delegation and acceptance, not spec production — the composition promised in the README's non-goals is now an actual rule.
- **The parallel gate must leave a table**: before dispatching writers concurrently, record one row per task pair sharing a file, compile/test target or interface. An unrecorded scan does not count as one. The resident gate's second condition now reads "no two write scopes share a compile/test target".

## [2026-09-04]

### Added
- **Review convergence.** A returned task's second pass verdicts each recorded finding `addressed` / `not addressed` ("attempted" is not addressed), scopes the fresh look to the fix diff, and records an out-of-scope observation separately — unless it breaks an acceptance scenario or a frozen contract, which blocks and replans. The loop is capped at five rounds, with a fresh worker from round four, and adjudication is permitted only at the cap.
- **Findings are routed by root cause.** Inside the frozen region ⇒ revert and take it to the user; outside it ⇒ the controller amends the spec/plan and re-dispatches; in the code alone ⇒ the worker patches. Code-level findings from the same round are moot once a spec-level route applies.
- **A review request may not pre-judge its verdict** ("don't flag X", "at most Minor", "the plan chose this"); a plan-mandated defect is still reported, labeled `plan-mandated`.
- **The freeze has a visible boundary.** Requirement intent, acceptance scenarios and contracts are wrapped in a greppable `<frozen-after-approval>` delimiter, so "is this edit inside the freeze?" stops being a question of memory.
- **Verification steps are runnable.** A dispatch contract carries `run:` / `expect:` pairs (a step that must fail first names the exact failure text); for a code-changing task `acceptance` and `verification` may not be empty, and a worker that cannot run a step returns `blocked` with the command and failure instead of a claim.
- **A green baseline is recorded before the first code-changing dispatch**, using the enforcement command bound during grounding; when it cannot be green, the already-failing checks are listed.
- **Cross-session state is explicit.** The ledger gained `request.mode`, `request.execute_authorized`, `analysis.baseline` and `tasks[].round`, plus a recovery protocol: after an interruption every `in_progress` task is unverified until its artifacts are re-checked, and authorization is re-checked because it belongs to the user's instruction, not to the session that received it.
- **New boundaries.** No writing task is dispatched onto a dirty or shared working tree (a `failed` task is reverted by its own `write_scope`, never by discarding the tree); widening a write scope or adding work beyond the confirmed spec needs the user's confirmation, as narrowing already did; `challenge` is read-only like `analyze` and `diagnose`.
- **`challenge` covers an existing implementation**, so a code or PR review has a mode with severity grading and evidence instead of free-form commentary.
- **Limitations & non-goals** in both READMEs: the rules are an observable check rather than a block (enforcement would have to live in tooling), resident cost is real, the parallel gate is a criterion and not a lock, the ledger is a file with no arbiter; and the non-goals — no task store/board/dashboard, no hosted state, no spec generation of its own.
- **Contribution governance.** A rule change states its gap, consequence class (`irreversible` / `one wasted round` / `noise`), where the cost lands, and its retirement condition; `Retiring a rule` gives four delete triggers and must be run over the section being touched before adding to it.

### Changed
- **The parallel gate is compile/test-target granular** in the resident boundary too, matching `decomposition.md`: different files inside one package/module are not independent.
- **`analyze` / `diagnose` / `challenge` are read-only with respect to product code and existing artifacts** — they still write their own deliverables and `.ai-work/` state. Neither a platform permission nor the implementation intent of the triggering request authorizes edits.
- **Control-loop steps link the references whose rules they enforce.** A reachability audit found steps 2, 4 and 7 with no links and `ledger.md` linked from no step, leaving the recovery protocol, the authorization re-check, the scope-widening gate, bug triage, the enforcement-command binding and pilot-then-batch reachable only through a conditional routing sentence.
- **Read-back covers a single-line edit**, not only multi-artifact batches: re-read the touched range or the diff, and re-ground line numbers before the next edit in the same file, because a line-addressed edit can land on a shifted line and still report success.
- **The capability knowledge base is optional**, with prior art coming from the repo and installed skills when none is configured; recording a capability that was not actually found is forbidden.
- **The frontmatter description states trigger conditions only**, no longer a process summary that contradicted the body's own "single-root-cause task, no delegation".
- **Rule force is graded where it matters**: the three harm/correctness invariants are capitalized (`MUST` be grounded in a run, `NEVER` fabricate a value, `MUST NOT` rewrite unreconstructable content) and rule-authoring guidance moved out of resident context to `CONTRIBUTING.md`.
- **Controller review checks worker self-consistency**: declared status/flags disagreeing with the report body is treated as a hallucination signal, not a clerical slip.
- **Anti-fabrication covers two hand-authored values**: timestamps come from the system clock, progress counts are derived by counting ledger states.
- **Progress surface has one source and a fallback**: the driver is defined once in `SKILL.md`, and where a platform has no native task list the phased view lives in the ledger, echoed on each state change.
- **The ledger file is created directly** with its path stated, instead of interrupting to ask.
- **Both worked examples were rebuilt** against current rules; they had previously demonstrated file-level parallel reasoning and gone from `analyze`/`diagnose` straight into edits.
- Earlier in this cycle: `references/tech-design.md`, `references/deliverables.md` and `references/experience.md` were added (technical design, artifact layout, opt-in skill self-improvement); English README, `CONTRIBUTING.md`, CI and a PR template landed; `verification.md` gained document/diagram evidence, TDD discipline and a shippable-software gate; `context-grounding.md` bound conventions repo-first with tooling enforcement.

### Removed
- **Two resident lines no scenario could violate**: the self-judged "use native modes only when independently useful" clause and the Progress-surface restatement that `analyze`/`diagnose` modify no code. First use of `Retiring a rule`.

## [2026-09-02] — Initial public release

### Added
- `requirement-orchestrator` skill: spec-driven, single-controller multi-agent orchestration across Codex and Claude.
- `SKILL.md` (modes, control loop, progress surface, target-system preflight, non-negotiable boundaries) and the initial `references/*` set.
- Chinese README (`README.md`) with pain-point, environment/deps, install, and usage.
