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
- `references/agent-contract.md` — `Review dispatch`: a review request must not pre-judge its verdict ("don't flag X", "at most Minor", "the plan chose this"); a plan-mandated defect is still reported, labeled `plan-mandated`, and a finding believed wrong is answered with a verdict and evidence rather than suppressed in advance. (Adapted from `obra/superpowers` `subagent-driven-development` and `task-reviewer-prompt.md`.)

### Changed
- `SKILL.md` — rules now graded with RFC 2119 and framed hybrid (few `MUST NOT` boundaries + positive defaults), with an anti-checklist-sprawl note.
- `references/verification.md` — document/diagram artifact evidence, test-first (TDD) discipline, and a shippable-software completion gate.
- `references/decomposition.md` — parallel gate sharpened: the unit of independent validation is the toolchain's compile/test target, not the file.
- `references/context-grounding.md` — convention grounding: repo-first, ecosystem-fallback, enforced by tooling (formatter/linter/test), not a rigid schema fixed by tech stack.
- `references/ledger.md` — ledger persists at internal `.ai-work/ledger.md`, never under publishable `docs/`.
- `references/verification.md` — controller review adds a consistency check: a worker's declared status/flags must agree with its own report body; a mismatch is treated as a hallucination signal and blocks recording state until the artifacts are re-checked. (Adapted from `sdi2200262/agentic-project-management` `task-review.md`.)
- `references/agent-contract.md` — `verification:` entries are now `run:` / `expect:` pairs (a step that must fail first names the exact failure text); an expectation no third party could observe is not verification and is replaced or dropped. (Adapted from `obra/superpowers` `writing-plans`.)

## [2026-09-02] — Initial public release

### Added
- `requirement-orchestrator` skill: spec-driven, single-controller multi-agent orchestration across Codex and Claude.
- `SKILL.md` (modes, control loop, progress surface, target-system preflight, non-negotiable boundaries) and the initial `references/*` set.
- Chinese README (`README.md`) with pain-point, environment/deps, install, and usage.
