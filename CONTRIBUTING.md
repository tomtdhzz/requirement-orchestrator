# Contributing

Thanks for improving Requirement Orchestrator. It is a **methodology skill — Markdown only** (no runtime, no dependencies): the value is in clear, well-graded rules, not code.

## Ground rules

- **`SKILL.md` is authoritative.** `references/*` hold on-demand detail; `README*.md` are human guides. Keep them consistent; on conflict `SKILL.md` wins.
- **Grade rule force with RFC 2119** (`MUST` / `MUST NOT` / `SHOULD` / `MAY`, meaningful only in capitals) and use restraint: reserve `MUST`/`MUST NOT` for correctness or harm; leave preferences as `SHOULD`/`MAY`.
- **Hybrid framing.** Keep hard boundaries few and negative (in `Non-negotiable boundaries`), pair each with a positive "do instead", and express operating guidance positively. Negative-only rules are unreliable for agents.
- **Restraint over accretion.** Prefer grading or consolidating an existing rule over adding a new one. A PR that removes/merges rules while keeping coverage is very welcome. Aim for fewer, sharper rules — not a longer checklist.
- **Evidence, not speculation.** Add or change a rule only when a concrete run or clear failure mode motivates it, and state four things in the PR: the **gap** (what goes wrong without it); the **consequence class** if it fires — `irreversible` (finished work overwritten, unauthorized edit, external system written, plan lost with the session) / `one wasted round` / `noise`; where the **cost** lands — resident `SKILL.md` (paid every session), an extra agent round-trip (paid per task), or human attention (paid at every gate); and its **retirement condition** (see `Retiring a rule`). A `noise`-class rule that costs resident context is rejected; an `irreversible`-class one may pay all three. A rule with no stated way to die is how a "few, sharp" set becomes a checklist.

## Where things go

- A new capability/how-to → a `references/*.md`, loaded on demand (not dumped into `SKILL.md`).
- A hard invariant → a paired `MUST NOT` + "do instead" in `SKILL.md` → `Non-negotiable boundaries`.
- Project-specific lessons are **not** skill content — they live in a project's `.ai-work/lessons.md` (see `references/experience.md`). Promoting a lesson into this skill is an **opt-in, human-approved** step, committed separately.

## Retiring a rule

Adding is gated; removing must be too, or the set only grows. Before adding any rule, run this check over the section you are about to touch and remove what it catches — that keeps net size roughly flat instead of monotonically rising.

Delete a rule when any of these holds:

- it has **never been triggered** — the failure it guards has not occurred in three months or ten runs, whichever comes first (do not keep it "just in case");
- it **always needs an exception** ("this time is different") — the criterion is wrong, so rewrite it rather than bolting on a nuance clause;
- it has **never changed an outcome** — no decision, artifact, or verdict differs because of it; it only made the process longer;
- **you no longer follow it yourself** — that is evidence it has no observable failure, which is the same defect it would flag in a spec.

Removal is a normal, welcome PR: cite which condition it met. Do not soften a rule that met one of these — a weakened rule keeps the cost and loses the effect.

This section retires under its own first condition: if three months pass with no rule removed and no PR citing the admission frame, nobody is using it — delete it rather than leaving a governance ritual in place.

## Commit messages

Follow [Conventional Commits](https://www.conventionalcommits.org/): `type(scope): imperative summary`, then a body that **clearly states what changed and why** — one bullet per change, cite evidence. The subject is the shape; the body is where a reader learns what you actually did.

- **Types**: `feat`, `fix`, `docs`, `refs` (reference-file content), `ci`, `chore`, `refactor`. Scope optional (e.g. `docs(README)`, `refs(decomposition)`).
- **One concern per commit**; keep `SKILL.md` behavior changes separate from docs.
- A commit template lives at [`.gitmessage`](.gitmessage) — enable it once: `git config commit.template .gitmessage`.

Example:

```text
refs: sharpen parallel gate to compile/test-target boundary

- decomposition.md: the unit of independent validation is the toolchain target, not the file
- why: two agents editing the same Go package broke each other's `go test` (bili2go run)
```

## Before you open a PR

- Every cross-reference link resolves; every Markdown/diagram renders as a reader will see it (render Mermaid, don't eyeball).
- No secrets, tokens, emails, or absolute personal paths. Never commit `.ai-work/`.
- Keep commits focused; skill-behavior changes are separate commits from docs.
- Update `CHANGELOG.md` under `[Unreleased]`.

## PR checklist

- [ ] Motivated by real evidence (linked/described)
- [ ] Gap, consequence class (`irreversible` / `one wasted round` / `noise`) and cost location stated
- [ ] Retirement condition stated; the touched section was checked against `Retiring a rule`
- [ ] RFC 2119 force is appropriate; not everything is `MUST`
- [ ] Prefers grading/consolidation over net-new rules
- [ ] Links resolve, docs render, no secrets, no `.ai-work/`
- [ ] `CHANGELOG.md` updated
- [ ] Commit messages follow Conventional Commits with a what/why body
