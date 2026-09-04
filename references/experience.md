# Experience Loop (minimal)

Purpose: stop paying the same discovery/mistake tax twice on **recurring** tasks. Carry
forward the project/environment-specific hard facts from prior runs so a similar task starts
with them in hand. This is not a "memory feature" and not for general methodology — general
lessons belong in your competence and in the other references, never here.

Deliberately minimal: a plain appended file, full-read at start. No index, no retrieval
engine, no consolidation — those are deferred until proven necessary (see Grow-later).

## Where
`.ai-work/lessons.md` inside the project being worked on. Lessons are project-specific, so
they travel with the project, like the ledger.

## At task end (control-loop step 8, after completion)
Append only lessons that pass ALL of:
- **Project/environment-specific** — not general dev knowledge (if it's general, it's not a
  lesson here; it's either already known or belongs in a reference).
- **Likely to recur** — you will plausibly hit this again in this project.
- **Evidence-backed** — cite what actually happened; no fabrication (same provenance rule as
  the ledger).

Entry format (one line):
`- [type] <trigger keywords> — <lesson>. (evidence: <what happened>; <date>)`
type ∈ tool-constraint | env-fact | failure-lesson | decomposition-pattern | calibration.

If nothing qualifies, append nothing. Most runs add 0–2 entries.

## At task start (control-loop step 1)
If `.ai-work/lessons.md` exists, read it during context grounding. Treat entries as
**candidate facts to verify**, not gospel — they can go stale; confirm against current code
before depending on one (see [context-grounding.md](context-grounding.md)). A stale entry is
corrected or deprecated, not trusted blindly.

## Boundaries
- Never fabricate an entry to look productive; absence is fine.
- Never auto-edit SKILL.md or the references from lessons. Promoting a recurring lesson into
  the skill itself is a human decision, made explicitly.

## Skill improvement (opt-in, separate from project lessons)

Project lessons above are automatic and project-local. Improving the **skill itself**
(SKILL.md / references) from a run is a distinct, **opt-in** action — default off, never
automatic (see the boundary above).

- **Offer, don't impose.** At task end, if a run surfaced a lesson that is *general* (not
  project-specific) and *likely to recur across projects*, offer promoting it to the skill as
  an option. Apply only if the user chooses it; otherwise it stays in `.ai-work/lessons.md`.
- **Evidence-gated.** Promote only what this run actually demonstrated; no speculative rules.
- **Restraint over accretion.** Prefer grading an existing rule (SHOULD / MAY) or
  consolidating it over adding a new one; a genuine invariant becomes a paired MUST NOT +
  "do instead" in `Non-negotiable boundaries` (see the rule-force note in SKILL.md). Aim for
  fewer, sharper rules — not a growing checklist.
- **Human-approved, committed separately.** Surface the exact change (file + wording +
  RFC 2119 force) for an explicit OK, together with what the promotion admits: the gap, its
  consequence class, where the cost lands, and the condition under which the rule would be
  retired (`CONTRIBUTING.md` → `Retiring a rule`). Promotion is the main way rules enter, so
  it goes through the same gate as a hand-written one. After editing, verify it renders and
  links resolve ([deliverables.md](deliverables.md)); commit skill changes on their own.

## Grow-later (only on evidence, not speculation)
Add real retrieval (`kb.py`-style query by task signature) and consolidation
(Importance / Merge / Decay / Eviction) ONLY when `lessons.md` grows past ~30 entries AND
reading it whole becomes unwieldy. Kill criterion: if entries are rarely acted on across
several runs, stop appending — the loop is not paying off for this project.
