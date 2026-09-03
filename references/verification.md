# Verification and Acceptance Evidence

An acceptance scenario is met only with **observable evidence** from exercising the change,
not a worker's success summary. This defines what counts as proof, per task type, and how
the controller reviews it. It backs control-loop steps 6 and 8.

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

**Record a green baseline before the first dispatch.** Run the project's checks on the
untouched tree and put the result in the ledger. A red or unknown baseline makes every later
failure ambiguous — you cannot tell whether a worker broke it or found it broken. When the
baseline cannot be green, record exactly which checks already fail; a later run is then
compared against that list, not against zero.

## Controller review (worker results)

A worker submits `review`; only the controller records `completed`, after checking:

- the requested behavior and each acceptance scenario, against artifacts — not the summary;
- the authorized write boundary was not exceeded;
- dependency and frozen-contract compatibility;
- the specified verification actually ran, with evidence recorded in `tasks[].evidence`;
- no unintended changes or unresolved risks.
- the worker's own declared status and flags agree with its report body — an inconsistency
  there is a hallucination signal, not a clerical slip: re-check the artifacts before
  recording any state.

A failed or unverifiable result returns to `in_progress` with specific findings; it is
never marked `completed`. The controller may make a small correction only when it changes
no behavior, contract, or scope.

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
  extend this task's loop.

The loop ends when every prior finding is `addressed` and the fix diff introduces nothing
new. Then the controller applies the review above and records `completed`.

## Anti-fabrication

- Claims of test/build/run results must be grounded in an actual run. Unobserved claims are
  labeled inference, not fact.
- Do not assert a derived value (classification, pass/fail, risk) beyond its verified
  source; an unverified derivation is marked as such and its gap recorded
  (see [ledger.md](ledger.md) provenance).
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
