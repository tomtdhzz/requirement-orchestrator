# Freeze Before Fanout

## The claim

No frozen boundary, no parallelism.

Almost every agent-orchestration failure today is not a capability failure. It is **fanning out too early**: work is handed to two or more executors while the contract, the scope, and the facts are still moving. Everything that follows is a consequence of that one step — two writers edit the same compile target, each passes locally, the merge does not build; two implementations each code against the interface they guessed, and integration turns into wholesale rework; one executor treats another's speculation as fact and derives from it, until no conclusion can be traced back to anything.

A freeze is not process ceremony. It is the **precondition** for parallelism: once the three things hold still, running in parallel is safe; while they do not, the correct move is not to parallelize.

The three freezes are **Contract**, **Scope**, and **Fact**. Each is given below with its failure consequence and its observable criterion.

## The step nobody named

Existing frameworks optimize one end of the chain or the other.

- Front end: **is the spec any good?** spec-kit (133k★), OpenSpec and similar tools structure requirements, acceptance and contracts, solving "the input is vague".
- Back end: **is the completion claim honest?** `obra/superpowers` (281k★) and its `verification-before-completion` solve "the executor says it is done".

The step in between — **what must be held still after the spec is written and before the work is actually cut apart and dispatched** — has never been named by any of them. Fragments exist everywhere: BMAD has role boundaries, superpowers has an evidence gate, spec-kit produces contracts. Each fragment has been validated on its own, but nobody assembled them into one teachable thing, so nobody can walk down it item by item before fanning out.

Why this middle step is the expensive one: orchestration cost lands in three places — resident context (paid every session), agent round-trips (paid every task), human attention (paid at every gate). However good the spec is, a wrong fanout means re-dispatching tasks, re-spending round-trips, and re-occupying human attention: all three paid a second time. It is the one point on the chain where getting it wrong costs full price again.

This document invents no new concept. It gives a name to something already being done but never named, and states the criteria by which it can be observed to fail.

## The three freezes

Three things must hold still before fanout. Each has its own failure shape and its own way of being checked.

| Frozen object | Consequence of not freezing | Observable criterion | Where it lives here |
| --- | --- | --- | --- |
| **Contract** | Integration-time conflict, wholesale rework | Does the contract text already exist, and is it unchangeable until the tasks depending on it are done? | [../references/spec-driven.md](../references/spec-driven.md) |
| **Scope** | Silent overwrites, builds and tests destroying each other | Do the two tasks land on **different compile/test targets**? (Different files do not count.) | [../references/decomposition.md](../references/decomposition.md) |
| **Fact** | Hallucinated derivation, conclusion drift | Can every derived value (a classification, a judgment, a risk rating, a recommendation) point back to one verified fact? | [../references/verification.md](../references/verification.md) |

All three can be checked on the spot: the contract text either exists or it does not; two write scopes either share a compile target or they do not; a conclusion either points back to a fact or it does not. None of them depends on an adjective like "sufficient" or "clear" — deliberately, because an adjectival criterion can never be observed to fail.

Contract and Scope only need freezing when work is delegated. Fact is independent of delegation (see "When you do not need this").

The three most common fake freezes, all of which look like a freeze happened:

- The contract exists only in the conversation and was never written down — the next session cannot see it, so it is not frozen.
- Scope was split by file instead of by compile/test target — two files in one package are not two scopes.
- The "fact" came from another executor's summary rather than an actual read or run — a summary can be restated speculation.

## The control plane: Freeze → Fanout → Fan-in

Each of the three stages has one thing that cannot be skipped.

**Freeze.** All three freezes land. In an existing codebase there is one more step: re-ground the unknowns in stages — repository and branch state and the build baseline first, then code facts, and only then contracts. A freeze that skipped grounding freezes a guess.

**Fanout.** One controlling agent owns dispatch exclusively; executors do not dispatch to each other. Only tasks with a complete contract go out: goal, write scope, acceptance criteria, dependencies — missing one means not dispatching. Before writing to any system beyond the local working tree (a wiki, a ticketing system, a database, a remote API), run the preflight — write permission and its exact scope, rate limits and whether failures are silent, whether the result can be read back — established before dispatch, not discovered mid-batch.

**Fan-in.** Acceptance through an evidence gate: an executor may only submit for review, and only the controller may record completion; bulk changes are confirmed by independent read-back and a structural delta, never by the tool's self-reported success count.

**Control does not transfer** across the three stages: dispatch, acceptance, and replanning stay with the same controller. If any stage delegates the judgment to an executor, the records of the other two stop lining up — nobody can say who cleared which boundary.

**This stage claims no originality.** It reuses established practice: `obra/superpowers`' `verification-before-completion` holds this position with 281k★, and its Iron Law is `NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE`. The position taken here is explicit reuse rather than reinvention — Freeze Before Fanout claims the step **before fanout**, not the acceptance standard itself.

## One real run

2026-09-01, Claude Code, diagnosing a failed branch deployment on a Go project. Every detail below comes from that run's notes.

**The Fact freeze paid off.** The CI build stage reported three `undefined` errors. Three errors, three symbols — the most natural move is to split them into three bugs and three subtasks and fix them in parallel. What blocked that move was the root-cause criterion: this repository's axis is written as "bug root cause and repair path" — **root cause**, not symptom — which forces the question "are these three the same cause?" to be answered first. They were: the branch was two commits behind master, and another commit on the branch had rewritten that file in master's shape, referring to symbols that do not exist on this branch. Three symptoms, one root cause, one task.

Without that criterion: three tasks editing the same package in parallel, each "fixing" its own error message, none of them touching the actual cause, and the error reappearing under a different symbol.

**The Contract freeze paid off.** **Before** touching the conflict resolution, the process-global registration constraint around `prometheus.MustRegister` was identified: if same-named metrics from the two sides are not fully deduplicated, the process panics at startup. **This is not something the compiler can report** — compilation is clean and the failure happens at the moment of startup. It surfaced from the deliberate act of "identify shared-state constraints", which is the "process-global registrations" entry on the contract-freeze list.

Without it: conflicts resolved, build green, panic on deploy — and nothing at the scene points back to that merge.

**Stated honestly: that run dispatched no subagent at all**; the controller was a single session. So the case validates **the criteria themselves** — the root-cause criterion blocked a wrong split, the shared-state criterion caught a conflict the compiler cannot catch — not the benefit of parallel execution. The same notes also record the cost: criteria are not free. A rule requiring a reference document to be read "in every mode" was simply bypassed on that three-line compile error, because reading two documents was out of proportion to the task.

## When you do not need this

For a single task, in a single session, with no delegation, the Contract and Scope freezes degenerate into asking and answering your own question: with no second writer there is no boundary to police, and writing a "frozen contract" only you will read is pure overhead.

The one that still earns its cost is the Fact freeze. "A derived conclusion that cannot point back to a fact" has nothing to do with parallelism — the run above was a single session, and both of its wins came from criteria, not from parallelism.

The test is crude but sufficient: with one writer or fewer, do the Fact freeze only; with two writers or more, do all three, and do not fan out while one is missing. Treated as an unconditional ritual, the three freezes get bypassed on small tasks — a three-line compile error cannot carry a contract document, and forcing it costs the rules their credibility as a whole.

## Criteria you can test

This concept carries its own failure criteria. If Freeze Before Fanout were empty, you would observe it failing; conversely, any of the symptoms below means one of the freezes did not happen.

1. **Parallel writers destroy each other.** Two executors each report "passes locally"; together they do not build or do not test green. → Scope was not frozen: two write scopes landed on the same compile/test target.
2. **Wholesale rework at integration.** Task-level acceptance is all green, then interfaces, field names, or error shapes do not line up, and rework is the same order of magnitude as the work already done. → Contract was not frozen: the contract was discovered mid-implementation instead of written down before fanout.
3. **Derived conclusions that cannot point back to a fact.** A classification, risk rating, or recommendation cannot be traced to a verified source, or traces to another executor's speculation. → Fact was not frozen.

All three are verifiable after the fact from the artifacts alone; none requires trusting anyone's account of the process. That is how to judge whether this document is useful, and equally how to judge whether your last fanout was done right.

The control loop itself is in [../SKILL.md](../SKILL.md); the specific rules for each freeze are in the "Where it lives here" column above.
