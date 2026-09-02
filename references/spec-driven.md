# Specification Model

A specification defines the **boundaries of a solution space**, not one implementation.
In `analyze` the deliverable is a spec at this shape; in `execute` every task is
implemented and verified against it. This is the content model behind the ledger's
`request`, `analysis`, `tasks[].acceptance`, and `integration.cross_task_contracts`.

## Three layers

Produce these three, in order. Each lower layer must trace to the one above.

1. **Requirements** — what must be true, in outcome terms, not implementation.
   - Each is observable and testable; no solution baked in ("users can reset a password",
     not "add a `/reset` endpoint").
   - Record non-goals explicitly. The boundary of the solution space is set by what is
     excluded as much as what is included.

2. **Acceptance scenarios** — concrete, checkable conditions that prove a requirement is
   met. Prefer Given/When/Then. Each scenario names its observable evidence
   (see [verification.md](verification.md)). These become `tasks[].acceptance` and the
   end-to-end `integration.final_verification`.

3. **Contracts** — the interfaces two parties agree on so work can proceed independently:
   API signatures, schemas, state semantics, identifiers, error shapes, event formats,
   process-global registrations. Contracts are what you **freeze** before dependent or
   parallel work starts (see [decomposition.md](decomposition.md) parallel gate). They
   become `integration.cross_task_contracts`.

## Rules

- **Trace, don't restate.** Every acceptance scenario points to a requirement; every task
  points to the scenarios it satisfies. An orphan task (no requirement) is scope creep; a
  requirement with no scenario is unverifiable and blocks the spec.
- **Contracts precede implementation.** If two tasks touch a shared boundary, write the
  contract first and freeze it. Unfrozen shared contract ⇒ tasks are not parallel-safe.
- **The spec is falsifiable.** If nothing could fail it, it says nothing. Every requirement
  needs at least one scenario whose failure would be observable.
- **Reduce scope only with explicit user approval;** never silently narrow requirements to
  fit a plan. Record the narrowed boundary in `analysis.decisions`.
- **Self-authored specs are reviewed before implementation.** When you draft the spec from
  a high-level prompt, treat planning and execution as separate steps: surface the spec
  (requirements + scenarios + contracts) for confirmation before dispatching writes.

## Where it enters the control loop

- Step 1 (establish goal/scope/acceptance): produce Requirements + Acceptance scenarios.
  Ground unknowns first via [context-grounding.md](context-grounding.md).
- Step 3 (decompose): derive Contracts, freeze shared ones, then cut tasks so each owns a
  slice of the spec with independent acceptance.
- Step 8 (complete): the request is done only when every acceptance scenario passes, not
  when tasks report done.

## Anti-patterns

- Implementation-shaped "requirements" that prejudge the solution.
- Acceptance written as "it works" — not observable, not checkable.
- Contracts discovered mid-implementation instead of frozen up front (the top cause of
  rework and parallel-write conflicts).
