# Changelog

All notable changes to this project are documented here.
The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/);
this project uses date-based entries (no semantic version tags yet).

## [Unreleased]

### Added
- `references/tech-design.md` — technical-design (the "how") reference: industry TDD/RFC structure, architecture-reflects-domain rule, consumer-interface contract rule, review checklist.
- `references/deliverables.md` — artifact layout: publishable docs (`docs/prd`, `docs/tech-design`) vs internal `.ai-work/` (ledger/plan/lessons); publishable-project scaffold (README/LICENSE/.gitignore) with runnable-command and copy-safe-placeholder rules.
- `references/experience.md` — opt-in skill self-improvement loop (default off, evidence-gated, human-approved, committed separately), distinct from automatic project lessons.
- Non-negotiable boundaries for deliverable self-review (render docs, run README commands as written) and publishable projects (no committed `.ai-work/`).
- English README (`README.en.md`), `CONTRIBUTING.md`, and this changelog.
- GitHub Actions CI (`.github/workflows/ci.yml`): internal Markdown link check + guard against committing `.ai-work/`.
- Pull request template (`.github/PULL_REQUEST_TEMPLATE.md`) and commit-message convention (Conventional Commits + `.gitmessage` + CONTRIBUTING section) requiring a what/why body.
- `references/verification.md` — `Re-review of returned work`: a returned task's second pass verdicts each prior finding `addressed` / `not addressed` ("attempted" is not addressed), scopes the fresh look to the fix diff, and records an out-of-scope observation separately instead of extending the loop. (Convergence protocol adapted from `obra/superpowers` `re-review-prompt.md`.)
- `references/verification.md` — `Review dispatch`: a review request must not pre-judge its verdict ("don't flag X", "at most Minor", "the plan chose this"); a plan-mandated defect is still reported, labeled `plan-mandated`, and a finding believed wrong is answered with a verdict and evidence rather than suppressed in advance. `agent-contract.md` keeps a one-line pointer. (Adapted from `obra/superpowers` `subagent-driven-development` and `task-reviewer-prompt.md`.)
- `references/spec-driven.md` — `Mark what is frozen`: the frozen material (requirement intent, acceptance scenarios, contracts) is wrapped in a greppable `<frozen-after-approval>` delimiter, so "is this edit inside the freeze?" and "is this finding's root cause inside it?" stop being questions of memory. Skipped for a single-root-cause fix with no delegation. (Adapted from `bmad-code-org/BMAD-METHOD` `spec-template.md`.)
- `references/verification.md` — `Routing a finding`: the root cause's position decides who fixes it — inside the frozen region ⇒ revert and take it to the user; outside it ⇒ the controller amends the spec/plan and re-dispatches; in the code alone ⇒ the worker patches. Code-level findings from the same round are moot when either of the first two exists. (Adapted from `bmad-code-org/BMAD-METHOD` `step-04-review.md`.)
- `references/verification.md` — the re-review loop is capped at five rounds per task, with a fresh worker from round four, and adjudication permitted **only at the cap** (ending a loop early by declaring a finding acceptable is pre-judging under another name); at the cap the task is parked and surfaced, never marked `completed` on the ruling alone. (Adapted from `obra/superpowers` `subagent-driven-development`.)
- `CONTRIBUTING.md` — rule admission and retirement governance: a rule change must state its gap, **consequence class** (`irreversible` / `one wasted round` / `noise`), where the **cost** lands (resident `SKILL.md` / agent round-trip / human attention), and its **retirement condition**; a new `Retiring a rule` section gives four delete triggers (never triggered · always needs an exception · never changed an outcome · you stopped following it) and requires running the check over the section being touched before adding to it. PR checklist and `references/experience.md`'s promotion path (the main way rules enter) route through the same gate.

### Changed
- `SKILL.md` — rules now graded with RFC 2119 and framed hybrid (few `MUST NOT` boundaries + positive defaults), with an anti-checklist-sprawl note.
- `references/verification.md` — document/diagram artifact evidence, test-first (TDD) discipline, and a shippable-software completion gate.
- `references/decomposition.md` — parallel gate sharpened: the unit of independent validation is the toolchain's compile/test target, not the file.
- `references/context-grounding.md` — convention grounding: repo-first, ecosystem-fallback, enforced by tooling (formatter/linter/test), not a rigid schema fixed by tech stack.
- `references/ledger.md` — ledger persists at internal `.ai-work/ledger.md`, never under publishable `docs/`.
- `references/verification.md` — controller review adds a consistency check: a worker's declared status/flags must agree with its own report body; a mismatch is treated as a hallucination signal and blocks recording state until the artifacts are re-checked. (Adapted from `sdi2200262/agentic-project-management` `task-review.md`.)
- `references/agent-contract.md` — `verification:` entries are now `run:` / `expect:` pairs (a step that must fail first names the exact failure text); an expectation no third party could observe is not verification and is replaced or dropped. (Adapted from `obra/superpowers` `writing-plans`.)
- `SKILL.md` — read-only boundary sharpened: the implementation intent of the triggering request does not carry into edits ("add login for me" authorizes analyzing that request, not editing in the same response); READMEs (zh/en) updated to match. (Adapted from `Fission-AI/OpenSpec` `workflows/propose.ts` planning boundary.)
- `references/mutation.md` — retitled `Safe Mutation` and widened: read-back is no longer scoped to multi-artifact batches. A single-line edit must be verified by re-reading the touched range or the diff (the intended lines changed, no adjacent line altered or dropped), and line numbers must be re-grounded before the next edit in the same file, because a line-addressed edit can land on a shifted line and still report success. `SKILL.md` routing line and READMEs synced.
- `references/verification.md` — a green baseline is recorded in the ledger before the first dispatch **that changes code**, using the enforcement command bound during grounding; when it cannot be green, the already-failing checks are listed so later runs compare against that list rather than zero. `agent-contract.md` carries the pointer so the rule is read at dispatch time. (Adapted from `obra/superpowers` `using-git-worktrees`.)
- `references/verification.md` — anti-fabrication extended to two hand-authored values: timestamps come from the system clock and progress counts (`N/M`, percentages) are derived by counting ledger states, never written by hand. (Adapted from `automazeio/ccpm` `conventions.md`.)
- `references/agent-contract.md` — its `Controller review` list is explicitly the short form of `verification.md` (authoritative checklist, re-review protocol, evidence standards), so the two cannot drift into competing sources; README (zh) on-demand index updated to surface re-review, the baseline, and review dispatch.
- `references/examples/*` — both worked runs rebuilt against current rules: they had demonstrated file-level parallel reasoning (contradicting the compile/test-target gate) and had gone from `analyze`/`diagnose` straight into edits with no authorization gate; they now show the gate, a recorded green baseline, `run`/`expect` verification steps, task `status`, the phased TODO, and self-review discipline when there is no delegation.
- `SKILL.md` / `references/knowledge-base.md` — the capability knowledge base is now explicitly optional with a stated fallback (repo + installed skills as prior art), instead of an unconditional control-loop step pointing at an integration that needs an external clone, `$RADAR`, and `python3`; recording a capability that was not actually found is forbidden.
- `SKILL.md` — the resident parallel gate now matches `references/decomposition.md`: writers may run in parallel only when no two write scopes share a compile/test target (different files inside one package/module are not independent). The two copies had drifted, and `CONTRIBUTING.md` makes the resident one authoritative — so the stale criterion was winning.

## [2026-09-02] — Initial public release

### Added
- `requirement-orchestrator` skill: spec-driven, single-controller multi-agent orchestration across Codex and Claude.
- `SKILL.md` (modes, control loop, progress surface, target-system preflight, non-negotiable boundaries) and the initial `references/*` set.
- Chinese README (`README.md`) with pain-point, environment/deps, install, and usage.
