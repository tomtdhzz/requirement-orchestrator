# Contributing

Thanks for improving Requirement Orchestrator. It is a **methodology skill — Markdown only** (no runtime, no dependencies): the value is in clear, well-graded rules, not code.

## Ground rules

- **`SKILL.md` is authoritative.** `references/*` hold on-demand detail; `README*.md` are human guides. Keep them consistent; on conflict `SKILL.md` wins.
- **Grade rule force with RFC 2119** (`MUST` / `MUST NOT` / `SHOULD` / `MAY`, meaningful only in capitals) and use restraint: reserve `MUST`/`MUST NOT` for correctness or harm; leave preferences as `SHOULD`/`MAY`.
- **Hybrid framing.** Keep hard boundaries few and negative (in `Non-negotiable boundaries`), pair each with a positive "do instead", and express operating guidance positively. Negative-only rules are unreliable for agents.
- **Restraint over accretion.** Prefer grading or consolidating an existing rule over adding a new one. A PR that removes/merges rules while keeping coverage is very welcome. Aim for fewer, sharper rules — not a longer checklist.
- **Evidence, not speculation.** Add or change a rule only when a concrete run or clear failure mode motivates it. Cite it in the PR.

## Where things go

- A new capability/how-to → a `references/*.md`, loaded on demand (not dumped into `SKILL.md`).
- A hard invariant → a paired `MUST NOT` + "do instead" in `SKILL.md` → `Non-negotiable boundaries`.
- Project-specific lessons are **not** skill content — they live in a project's `.ai-work/lessons.md` (see `references/experience.md`). Promoting a lesson into this skill is an **opt-in, human-approved** step, committed separately.

## Before you open a PR

- Every cross-reference link resolves; every Markdown/diagram renders as a reader will see it (render Mermaid, don't eyeball).
- No secrets, tokens, emails, or absolute personal paths. Never commit `.ai-work/`.
- Keep commits focused; skill-behavior changes are separate commits from docs.
- Update `CHANGELOG.md` under `[Unreleased]`.

## PR checklist

- [ ] Motivated by real evidence (linked/described)
- [ ] RFC 2119 force is appropriate; not everything is `MUST`
- [ ] Prefers grading/consolidation over net-new rules
- [ ] Links resolve, docs render, no secrets, no `.ai-work/`
- [ ] `CHANGELOG.md` updated
