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

### Changed
- `SKILL.md` — rules now graded with RFC 2119 and framed hybrid (few `MUST NOT` boundaries + positive defaults), with an anti-checklist-sprawl note.
- `references/verification.md` — document/diagram artifact evidence, test-first (TDD) discipline, and a shippable-software completion gate.
- `references/decomposition.md` — parallel gate sharpened: the unit of independent validation is the toolchain's compile/test target, not the file.
- `references/context-grounding.md` — convention grounding: repo-first, ecosystem-fallback, enforced by tooling (formatter/linter/test), not a rigid schema fixed by tech stack.
- `references/ledger.md` — ledger persists at internal `.ai-work/ledger.md`, never under publishable `docs/`.

## [2026-09-02] — Initial public release

### Added
- `requirement-orchestrator` skill: spec-driven, single-controller multi-agent orchestration across Codex and Claude.
- `SKILL.md` (modes, control loop, progress surface, target-system preflight, non-negotiable boundaries) and the initial `references/*` set.
- Chinese README (`README.md`) with pain-point, environment/deps, install, and usage.
