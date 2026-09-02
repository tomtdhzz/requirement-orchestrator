# Design note — the experience loop (why it's minimal)

Rationale behind [`references/experience.md`](../references/experience.md). Operational rules
live there; this explains the design choices for anyone extending the skill.

## Purpose (single, falsifiable)
Stop paying the same discovery/mistake tax twice on **recurring** tasks: a similar task
should start with the project/environment-specific facts and pitfalls learned last time. It
is not a "memory feature" and not for general methodology — general lessons belong in the
other references or in base competence.

## Why minimal (append-only file, full read at start)
The dominant failure mode of agent memory is bloat: "an agent that remembers everything
remembers nothing useful." So the design is deliberately the smallest thing that can prove
the purpose:
- **Append** grounded, project-specific lessons at task end; **read the whole file** at task
  start. No index, no retrieval engine, no consolidation.
- Infrastructure (query-by-signature retrieval, and consolidation via
  Importance / Merge / Decay / Eviction) is deferred until the log outgrows a whole-read.
- A **kill criterion** is built in: if entries are rarely acted on across several runs, stop
  appending — the loop is not paying off here.

This mirrors 2026 practice while avoiding premature infrastructure:
- procedural memory as structured markdown ("skills in prompts") rather than a DB —
  Memento-Skills / MemP;
- consolidation's four levers as the *later* growth path — Hindsight;
- hindsight distillation of reusable lessons — RetroAgent;
- and it explicitly does **not** auto-rewrite the skill's own instructions (LangMem does;
  we keep promotion human-gated).

## Guardrails
- **Grounded, not fabricated.** Every entry cites what actually happened (same provenance
  rule as the ledger). Absence is fine; do not invent entries.
- **Candidate facts, not gospel.** Entries can go stale; verify against current code before
  depending on one.
- **Human-gated promotion.** Turning a recurring lesson into a permanent rule in the skill
  itself is an explicit human decision, never automatic.

## When to grow it
Only on evidence: the log passes ~30 entries AND whole-read becomes unwieldy AND entries are
demonstrably acted on. Then add retrieval + consolidation — reusing the same file-native,
offline approach, not a new dependency.

## References
- MemP — Exploring Agent Procedural Memory: <https://arxiv.org/html/2508.06433>
- Managing Procedural Memory in LLM Agents (Memento-Skills): <https://arxiv.org/pdf/2606.23127>
- The Consolidation Problem in Agent Memory (four levers): <https://hindsight.vectorize.io/blog/2026/05/21/agent-memory-consolidation>
- RetroAgent (retrospective distillation): <https://arxiv.org/html/2603.08561v4>
