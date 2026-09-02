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

No suitable runtime for the changed surface ⇒ verify with a behavioral/smoke test and state
explicitly that visual verification was not possible.

## Controller review (worker results)

A worker submits `review`; only the controller records `completed`, after checking:

- the requested behavior and each acceptance scenario, against artifacts — not the summary;
- the authorized write boundary was not exceeded;
- dependency and frozen-contract compatibility;
- the specified verification actually ran, with evidence recorded in `tasks[].evidence`;
- no unintended changes or unresolved risks.

A failed or unverifiable result returns to `in_progress` with specific findings; it is
never marked `completed`. The controller may make a small correction only when it changes
no behavior, contract, or scope.

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
