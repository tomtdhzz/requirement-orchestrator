# Verification and Acceptance Evidence

An acceptance scenario is met only with **observable evidence** from exercising the change,
not a worker's success summary. This defines what counts as proof, per task type, and how
the controller reviews it. It backs control-loop steps 5, 6, and 8.

## Evidence by task type

| Task type | What counts as proof |
| --- | --- |
| Experiment / investigation | The run itself; captured output is the deliverable. No test. |
| Bug fix | A reproduction that failed before, then no longer triggers after the fix. |
| Permanent feature / API change | The changed-contract test(s) pass; add a test only for a new observable contract. |
| Web UI change | Driven against the real surface (browser), visual/behavioral confirmation. |
| TUI / CLI change | The actual program launched, the changed path exercised, output observed. |
| Bulk / external mutation | Independent read-back + structural delta (see [mutation.md](mutation.md)), not the tool's count. |
| Document / diagram artifact | Rendered as the reader will see it — Markdown structure intact, Mermaid/code fences parse and render, tables/links resolve; not just "the file was written". |

No suitable runtime for the changed surface ⇒ verify with a behavioral/smoke test and state
explicitly that visual verification was not possible.

**Test-first for permanent features (TDD).** Prefer building behavioral units test-first:
write the failing test (red) and run it to confirm it fails for the right reason, then
implement to green, then refactor. The red run is itself evidence the test can fail — a test
that never failed proves nothing. Pure data types and thin adapters may skip the red step,
but any non-trivial logic (selection, parsing, orchestration) is built red→green→refactor.

**Record a green baseline before the first dispatch that changes code.** Run the
enforcement command bound during grounding — formatter / linter / tests
([context-grounding.md](context-grounding.md)) — on the untouched tree and put the result in
the ledger. A red or unknown baseline makes every later failure ambiguous: you cannot tell
whether a worker broke a check or found it broken. When the baseline cannot be green, record
exactly which checks already fail; a later run is then compared against that list, not
against zero.

## Controller review (worker results)

A worker submits `review`; only the controller records `completed`, after checking:

- the requested behavior and each acceptance scenario, against artifacts — not the summary;
- the authorized write boundary was not exceeded;
- dependency and frozen-contract compatibility;
- the specified verification actually ran, with evidence recorded in `tasks[].evidence`;
- no unintended changes or unresolved risks;
- the worker's own declared status and flags agree with its report body — an inconsistency
  there is a hallucination signal, not a clerical slip: re-check the artifacts before
  recording any state.

A failed or unverifiable result returns to `in_progress` with specific findings; it is
never marked `completed`. The controller may make a small correction only when it changes
no behavior, contract, or scope.

## Routing a finding

Where a finding's root cause sits decides who fixes it. Getting this wrong is what makes a
spec and its code diverge permanently.

- **Root cause inside the frozen region** ([spec-driven.md](spec-driven.md)) — the intent
  itself is wrong or missing: revert the change and take it to the user. The controller does
  not renegotiate frozen intent on its own.
- **Root cause outside the frozen region** — the intent holds but the plan derived from it
  does not (wrong module, wrong task boundary, mis-derived acceptance): the controller amends
  the spec or plan first, then re-dispatches. Patching only the code leaves the next task
  reading the same wrong plan and reintroducing the defect.
- **Root cause in the code alone:** return it to its worker for a small fix.

When a finding of the first two kinds exists, code-level findings from the same round are
moot — that code will be re-derived. Do not spend a round fixing them.

With no delegation the routing still holds; only the actor collapses. You revert and ask
yourself the same question — intent, plan, or code — before touching anything.

## Review dispatch

A review is worth only its independence. Do not pre-judge the verdict in the request:
wording such as "don't flag X", "at most Minor", "treat this as intentional", or "the plan
chose this" removes a finding before it can be made, and leaves an evidence gate that cannot
fail. If you are about to write one of those, stop — you are deciding the outcome, not
requesting a review.

Do instead: give the change, the contract it must satisfy, and the evidence paths, then let
the reviewer produce findings. A defect the plan or the contract mandated is still reported,
labeled `plan-mandated`; the plan does not grade its own work. Answer a finding you believe
is wrong with a verdict and evidence — the re-review protocol below — never by suppressing
it in advance.

## Re-review of returned work

A returned task's second pass **verdicts the recorded findings**; it does not restart the
review. Without this scoping, findings multiply each round and the review never converges.

- **Verdict every prior finding** as `addressed` or `not addressed`, one line each. Silently
  dropping or merging a finding is a defect in the review itself, not a shortcut.
- **"Attempted" is not addressed.** A code change in the right area, a partial fix, or a
  stated intention does not clear a finding — the specific defect must no longer exist.
- **Scope the fresh look to the fix diff:** what the fix itself broke. Do not re-audit code
  the fix did not touch; that ground was covered in the first pass.
- **An out-of-scope observation is recorded, not blocking.** Note it as its own finding
  (ledger evidence, or a deferred item) and let it be scheduled separately. It does not
  extend this task's loop — **unless it breaks an acceptance scenario or a frozen contract**,
  which blocks and goes back through replanning (control-loop step 7). Without that
  exception, the round in which a defect surfaced would decide whether it blocks: the same
  silent-data-loss finding would hold the task in the first pass and be deferrable in the
  second.

The loop ends when every prior finding is `addressed` and the fix diff introduces nothing
new. Then the controller applies the review above and records `completed`.

Cap the loop at five rounds per task and record the round count with the task. A repeated
failure is a signal about the task, not about worker effort: from round four, dispatch a
fresh worker (or a more capable model) rather than asking the same one to retry unchanged.

**Adjudicate only at the cap.** Deciding earlier that a finding is acceptable, in order to
end the loop, is pre-judging under another name (see Review dispatch above). At the cap,
record the decision and what it costs if wrong, park the task, and surface it to the user —
never mark it `completed` on the strength of the ruling alone.

## Anti-fabrication

- Claims of test/build/run results MUST be grounded in an actual run. Unobserved claims are
  labeled inference, not fact.
- Do not assert a derived value (classification, pass/fail, risk) beyond its verified
  source; an unverified derivation is marked as such and its gap recorded
  (see [ledger.md](ledger.md) provenance).
- Two values are never authored by hand: a timestamp comes from the system clock, and a
  progress count (`N/M`, a percentage) is derived by counting ledger states. Both look
  harmless when invented and both silently misreport how much is verified.
- "Done" means the specified end-to-end behavior plus every named acceptance scenario — not
  a compiling scaffold, a narrowed test, or a plausible subset.

## End-to-end gate (step 8)

Declare the request complete only when task-level checks, every frozen cross-task contract,
and all acceptance scenarios pass together. Record the end-to-end result in
`integration.final_verification`.

For shippable software, "done" also requires, before declaring complete:

- non-trivial behavioral units were built test-first (TDD, above);
- every generated doc renders and every README/usage command runs as written
  (see [deliverables.md](deliverables.md));
- a standalone project is publishable — README, LICENSE, `.gitignore` — with internal
  `.ai-work/` artifacts excluded, not committed.
