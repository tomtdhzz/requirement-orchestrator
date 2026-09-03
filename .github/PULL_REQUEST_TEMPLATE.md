<!-- Keep it concise. Delete sections that don't apply. Comments (<!-- -->) don't render. -->

## What & why
<!-- What does this change do, and why is it needed? A few sentences. -->

## Change type
<!-- check all that apply -->
- [ ] `refs` — reference content (`references/*`)
- [ ] `docs` — README / CONTRIBUTING / CHANGELOG / comments
- [ ] `feat` / `fix` — behavior change to `SKILL.md` rules or the control flow
- [ ] `ci` / `chore` — workflows, tooling, maintenance

## Evidence
<!-- What concrete run or failure mode motivates this? Link or describe it.
     Rules change on evidence, not speculation. -->

## Rule hygiene (for SKILL.md / references changes)
- [ ] RFC 2119 force is appropriate — not everything is `MUST`
- [ ] Prefers grading / consolidating an existing rule over adding a new one
- [ ] Any hard invariant is a paired `MUST NOT` + "do instead" in `Non-negotiable boundaries`

## Pre-merge checklist
- [ ] Internal links resolve and Markdown/diagrams render (CI green)
- [ ] No secrets, tokens, emails, or absolute personal paths; `.ai-work/` not committed
- [ ] Commits follow Conventional Commits with a body stating **what & why** (see `CONTRIBUTING.md`)
- [ ] `CHANGELOG.md` updated under `[Unreleased]`

## Related issues
<!-- e.g. Closes #123 -->
