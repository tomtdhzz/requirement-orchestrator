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

## Grow-later (only on evidence, not speculation)
Add real retrieval (`kb.py`-style query by task signature) and consolidation
(Importance / Merge / Decay / Eviction) ONLY when `lessons.md` grows past ~30 entries AND
reading it whole becomes unwieldy. Kill criterion: if entries are rarely acted on across
several runs, stop appending — the loop is not paying off for this project.
