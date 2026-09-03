# Requirement Ledger

The ledger is the shared source of truth across agents, sessions, Codex, and Claude. Preserve decisions and evidence, not hidden reasoning or full chat transcripts.

## Persistence

- A small, single-session task may keep the ledger in the active conversation.
- Persist it for multi-agent, cross-session, or cross-platform work.
- Prefer an existing project task location. Otherwise use `.ai-work/ledger.md` (or `.ai-work/tasks/<slug>/ledger.md` when several tasks run concurrently); confirm before creating it.
- In an adopted Trellis flow, map the ledger into `.trellis/tasks/` rather than maintaining a competing task store.
- The ledger is internal orchestration state, not publishable docs — never under a published `docs/` path; for a publishable project `.ai-work/` is gitignored (see [deliverables.md](deliverables.md)). The YAML format below is unchanged.

Use readable Markdown with one controlled YAML block for machine state:

```yaml
request:
  goal: ""
  scope: []
  acceptance: []
  constraints: []

analysis:
  facts: []
  assumptions: []
  decisions: []
  gaps: []
  risks: []

tasks:
  - id: T1
    goal: ""
    decomposition_axis: feature
    depends_on: []
    read_scope: []
    write_scope: []
    acceptance: []
    status: pending
    assignee: null
    evidence: []

integration:
  cross_task_contracts: []
  unresolved_conflicts: []
  final_verification: []
  next_action: ""
```

## Derived output carries provenance

A derived artifact — a classification, label, summary, risk rating, or recommendation — is only as trustworthy as the fact it rests on. For each derived claim, record which verified fact in `analysis.facts` supports it. Where the supporting fact is unverified, mark the derived item as unverified rather than asserting it, and place the open question in `analysis.gaps`. Never fabricate a value to fill a slot; absence of evidence is reported as a gap, not smoothed over.

## State transitions

```text
pending -> in_progress -> review -> completed
                      \-> blocked
                      \-> failed
review -> in_progress
```

Workers report `review`, `blocked`, or `failed`. The controller alone records `completed` after verification.

## Handoff snapshot

Before changing the controlling platform, append a snapshot containing:

- current mode and phase;
- confirmed decisions and open gaps;
- running tasks and their owners;
- blocked and unverified results;
- current write scopes and frozen contracts;
- the single next action.

The new controller must acknowledge the snapshot before dispatching. The old controller then stops scheduling work.

