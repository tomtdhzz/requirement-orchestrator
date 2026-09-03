# Context Grounding

Multi-step work in an existing codebase fails less from bad logic than from **missing or
stale context**: unknown architecture, assumed-but-wrong dependencies, and mismatched
conventions. These compound across planning → decomposition → implementation. Ground the
context **before** you decompose, and record it as verified facts.

Do this in control-loop step 1, before writing the spec's contracts. It is read-only.

## Gather, then record as `analysis.facts`

- **Architecture reality:** the actual modules/services the change touches, how they call
  each other, where the seam for this change is. Not the diagram — the code.
- **Dependencies:** the versions and APIs actually in use (not assumed). Confirm a symbol
  exists and its signature before a task depends on it.
- **Conventions (repo-first, ecosystem-fallback):** how this repo already does the thing —
  naming, error handling, test layout, config — matching the existing pattern rather than
  introducing a third. For a **greenfield** project with no conventions yet, adopt the
  **ecosystem's** instead: the language's idioms and standard project layout, plus its
  standard formatter / linter / type-checker / test runner. On conflict, repo conventions
  win over generic ecosystem defaults.
- **Prior art:** an existing capability, helper, or skill that already solves part of this
  (consult [knowledge-base.md](knowledge-base.md) — prefer reuse over rebuild).
- **Blast radius:** every caller/consumer of what you'll change (for exported symbols,
  enumerate references before planning the edit).

## Rules

- **Investigate before asking.** Exhaust code, config, history, and the knowledge base;
  ask the user only for decisions that materially change scope or direction.
- **Record provenance.** Each grounded fact enters `analysis.facts`; anything you could not
  confirm enters `analysis.gaps` and is treated as an assumption, not a fact.
- **Assumptions are liabilities.** A task built on an unverified assumption is not
  parallel-safe and its acceptance is suspect. Convert assumptions to facts or to gaps.
- **Bind conventions; enforce with tooling, not prose.** Record the chosen conventions and
  the enforcement command (formatter / linter / vet / tests) as constraints, pass both to
  every worker in its contract, and make passing that command part of acceptance — do not
  hand-police style. Conventions are facts to verify and correct, **not a rigid schema fixed
  by tech stack**: a paradigm (e.g. DDD vs simple layered) is a design decision sized to the
  problem, not an automatic consequence of the language (match ceremony to the problem —
  see [tech-design.md](tech-design.md)).
- **Delegate discovery when the surface is unknown.** Use a read-only scout to map affected
  code rather than opening files hoping; bring back a compressed fact set, not raw dumps.
- **Re-ground on surprise.** If a worker reports the code differs from a recorded fact,
  the fact was stale: update `analysis.facts` and recompute the affected branch
  (see [decomposition.md](decomposition.md) replanning).

## Failure modes this prevents

- Planning against an architecture that no longer exists.
- A task depending on an API/signature that was never there or has changed.
- Introducing a third convention beside two existing ones.
- Missing a callsite of a changed contract and shipping a break.
