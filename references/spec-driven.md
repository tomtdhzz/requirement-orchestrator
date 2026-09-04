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

- **Never create a second spec source.** If the repository already holds the spec — a
  framework's directory, a wiki page, an issue — read it and work from it; this skill owns
  delegation and acceptance, not spec production. Record where it lives in the ledger's
  `analysis.decisions` so a later session knows why there is no `docs/prd/` here. A spec
  that is thin (a requirement with no scenario, acceptance written as "it works") is a
  finding to raise, not a reason to rewrite it in this layout.
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

## Mark what is frozen

A freeze is only enforceable if its boundary is visible. In the spec and the design, wrap the
frozen material — requirement intent, acceptance scenarios, contracts — in a delimiter that
survives a grep:

```markdown
<frozen-after-approval reason="human-owned intent — renegotiate with the user, do not edit">
… requirements · acceptance scenarios · contracts …
</frozen-after-approval>
```

Everything outside it — task breakdown, file layout, sequencing, implementation notes — is
the plan's argument about how to satisfy the frozen part, and the controller may amend it.
Two questions then become mechanical instead of remembered: whether an edit lands inside the
frozen region, and whether a review finding's root cause sits inside it (the routing in
[verification.md](verification.md)).

Applies when work is delegated or spans sessions. Skip it for a single-root-cause fix you
carry out yourself: with one agent and no worker, there is no boundary to police.

## Where it enters the control loop

- Step 1 (establish goal/scope/acceptance): produce Requirements + Acceptance scenarios.
  Ground unknowns first via [context-grounding.md](context-grounding.md).
- Step 3 (decompose): derive Contracts, freeze shared ones, then cut tasks so each owns a
  slice of the spec with independent acceptance.
- Step 8 (complete): the request is done only when every acceptance scenario passes, not
  when tasks report done.
- After the spec: for a non-trivial build, produce the technical design ([tech-design.md](tech-design.md)) and persist artifacts per the `docs/` layout ([deliverables.md](deliverables.md)). The spec is the *what*; the design is the *how*.

## Anti-patterns

- Implementation-shaped "requirements" that prejudge the solution.
- Acceptance written as "it works" — not observable, not checkable.
- Contracts discovered mid-implementation instead of frozen up front (the top cause of
  rework and parallel-write conflicts).
