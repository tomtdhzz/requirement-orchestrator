# Technical Design Document (the "how")

[spec-driven.md](spec-driven.md) defines the **spec** — what must be true (requirements),
how it is proven (acceptance scenarios), and the interfaces parties agree on (contracts).
This defines the **technical design**: how the solution is actually built — the industry
TDD/RFC artifact. Produce it in `analyze`, after the spec and before `execute`, for any
non-trivial build. Skip it for a single-root-cause fix or a change small enough that the
spec plus a task list is sufficient.

## Relationship to the spec

- **Spec** (spec-driven.md) = WHAT + acceptance + frozen contracts.
- **Tech design** = HOW: architecture, module/data/interface design, alternatives, and
  cross-cutting concerns.
- Do not repeat the spec; reference it (a brief goals/non-goals summary is fine). Contracts
  frozen in the design feed the ledger's `integration.cross_task_contracts` and the
  decomposition parallel gate.

## Sections

Right-size to complexity. Drop sections that do not apply; for a moderately complex change
most of the detail is thin. For complex work with many dependencies and risks, fill it out.

0. **Doc info** — author, status, version, date, linked PRD/ledger/plan, reviewers.
1. **Background & problem statement** — why now; the constraint that makes this non-trivial.
2. **Goals & non-goals** — summary, referencing the spec (do not restate requirements).
3. **Glossary** — domain terms and identifiers a new reader needs.
4. **Solution overview** — the chosen approach in a paragraph + an architecture diagram.
5. **Detailed design** — directory/module layout, data model, interface contracts (upstream
   dependencies AND the external API/CLI this exposes), sequence diagrams for core flows,
   and key algorithms.
6. **Alternatives considered & trade-offs** — the RFC core: the options weighed, what was
   rejected and why, and what breaks if the recommendation is wrong. Mandatory for every
   load-bearing decision.
7. **Cross-cutting concerns** — authentication/config, error handling, concurrency,
   resilience/networking, performance, observability, security.
8. **Testing strategy** — unit / integration / end-to-end, each mapped to the acceptance
   evidence it produces (see [verification.md](verification.md)).
9. **Rollout & milestones** — task breakdown, dependencies, and the parallel gate
   (see [decomposition.md](decomposition.md)). This is the design-level view; the live
   execution view is the plan doc + phased TODO.
10. **Risks & mitigations.**
11. **Open questions** — unresolved items, each tied to a gap in the ledger.
12. **References** — source anchors (files/line ranges), prior art, external templates.
13. **Revision history.**

## Rules

- **Falsifiable and specific.** The value of a design doc is forced specificity, not
  ceremony. If nothing in it could be wrong, it says nothing.
- **Trace decisions to requirements.** A design choice with no requirement behind it is
  scope creep; a load-bearing choice with no alternatives section is unreviewable.
- **Architecture reflects the domain and the language.** Model the ubiquitous language as
  first-class types; separate the domain from infrastructure (keep an anti-corruption
  boundary so transport/DTO shapes do not leak inward); follow the target language's idioms
  and standard project layout. Match ceremony to the problem — neither a technical-layer
  dump that mixes domain with I/O nor gratuitous full-DDD on a stateless tool.
- **Specify the consumer-facing interface, not just internals.** For anything another party
  calls — a frontend/HTTP API, a CLI, a library surface — give the exact contract:
  endpoints/commands, parameters, response schema with concrete examples, and status/error
  codes, plus the expected usage scenarios end to end. Acceptance verifies against this
  contract; an unspecified interface cannot be verified or built against in parallel.
- **Freeze contracts before dependent or parallel work** (decomposition parallel gate).
- **Self-authored designs are reviewed before execute.** Surface the design for
  confirmation; a native-plan approval or platform permission does not authorize edits.
- **Derived claims carry provenance.** A stated performance/behavior expectation that was
  not measured is marked `[INFERENCE]`, not asserted (same rule as verification.md).

## Where it enters the control loop

- Step 1/3 (`analyze`): after the spec, produce the tech design and freeze contracts.
- Persist it per [deliverables.md](deliverables.md) at `docs/tech-design/`.

## Anti-patterns

- A "design" that is only a task list with no architecture, data model, or alternatives.
- Restating the PRD instead of designing against it.
- Alternatives section omitted, so the reader cannot tell what was rejected or why.
- Cross-cutting concerns (auth, errors, concurrency, security) discovered during `execute`
  instead of designed up front — a top cause of rework.
- Domain vocabulary buried inside infrastructure/DTO types, so the design shows no domain.
- An exposed API/CLI described only by name, with no request/response shape, examples, or
  errors — leaving consumers and verification with nothing concrete to check.

## Review checklist (gate before leaving `analyze`)

Run this on a self-authored design before surfacing it for confirmation or entering
`execute`. A failed item is a defect to fix now, not a follow-up.

- [ ] Every requirement in the spec is served by a design element; no orphan design, no
  unaddressed requirement.
- [ ] Architecture is idiomatic to the language AND expresses the domain — ubiquitous
  language as first-class types, domain separated from infrastructure (anti-corruption
  boundary), standard project layout, ceremony matched to the problem.
- [ ] Every consumer-facing interface (API / CLI / library) has an exact contract:
  parameters, response schema with concrete examples, status/error codes, and end-to-end
  usage scenarios — the basis acceptance verifies against.
- [ ] Every load-bearing decision has an alternatives-and-trade-offs entry.
- [ ] Cross-cutting concerns (auth, errors, concurrency, resilience, security,
  observability) are addressed, not deferred to `execute`.
- [ ] Shared contracts to freeze for the decomposition parallel gate are identified.
- [ ] Acceptance scenarios map to concrete evidence.
- [ ] The document renders correctly — structure intact, diagrams parse, links resolve
  (see [deliverables.md](deliverables.md) self-review).
