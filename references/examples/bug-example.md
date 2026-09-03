# Example — Bug flow (diagnose → authorized → execute)

Illustrative end-to-end run of a reported failure. Shows reproduction-before-root-cause,
single root cause per repair path, and evidence-gated closure.

**Report:** "Checkout intermittently returns 500."

## 1. Reproduce before claiming a cause (diagnose, read-only)

Multiple 500s are symptoms until evidence shows independent causes
([decomposition.md](../decomposition.md) triage). Attempt a proportionate reproduction:
replay checkout under light concurrency → 500 reproduces ~1/20 requests. Capture a failing
trace: `NULL` inserted into `orders.coupon_id` when cart has no coupon.

Until reproduced, root-cause statements are **hypotheses**. Now it is reproduced, with a
trace, so the cause is evidence-backed.

Root cause (confirmed): handler passes `cart.coupon?.id` (undefined) to a NOT NULL column;
the intermittency is just how often carts lack a coupon.

Recorded facts vs gaps:
```yaml
analysis:
  facts:
    - "repro: ~1/20 checkout under concurrency -> 500 (trace attached)"
    - "orders.coupon_id is NOT NULL; handler sends undefined when no coupon"
  gaps:
    - "are there other NOT NULL columns fed from optional cart fields? (to check)"
```

## 2. Repository-state lens (if it were a build/deploy failure)

Not applicable here (logic defect, reproduced). For a "symbol suddenly missing" style
failure, check branch/HEAD/merge-base/divergence before assuming a code defect.

## 3. Authorization gate

`diagnose` ends here: the reproduced failure, the confirmed root cause with its trace, the
open gap, and the repair options are the deliverable. "Checkout intermittently returns 500"
is a report, not an authorization — and the absence of delegation does not change that. The
user picks a repair path and says to fix it; only that opens `execute`.

Baseline before the fix: `npx jest` → `318 passed, 0 failed` recorded in the ledger, so the
regression test's later red run is unambiguous.

## 4. Spec the fix (one root cause, one repair path)

- **Requirement:** checkout with no coupon succeeds; coupon path unchanged.
- **Acceptance scenarios:** A1 the reproduction (no-coupon checkout under concurrency) no
  longer 500s; A2 a valid-coupon checkout still records `coupon_id`; A3 regression covers
  the null path.
- **Contract:** none shared → single task, no freeze needed.

```yaml
tasks:
  - id: T1
    goal: "Fix null coupon_id on couponless checkout + regression test"
    decomposition_axis: bug
    depends_on: []
    write_scope: [src/checkout/handler.ts, test/checkout.spec.ts]
    acceptance: ["A1 repro no longer triggers", "A2 coupon path intact", "A3 test added"]
    verification:
      - run: "npx jest test/checkout.spec.ts -t 'no coupon'"
        expect: "FAIL — expected 201, got 500 (before the fix; this is the red run)"
      - run: "npx jest test/checkout.spec.ts"
        expect: "PASS — incl. 'no coupon' and 'valid coupon records coupon_id'"
      - run: "scripts/replay-checkout.sh --concurrency 4 --n 200"
        expect: "0 responses with status 500 (was ~10/200)"
    status: pending
```

Single root cause + single repair path ⇒ no delegation; the controller runs the loop itself.
That changes who does the work, not the gates: the mode and the authorization above still
apply, and the acceptance evidence below is the same standard. Fix source (make `coupon_id`
nullable or omit the field when absent — choose per schema intent), not the symptom (don't
swallow the 500).

Reviewing your own work removes the independent reviewer, not the review: judge the diff
against the frozen acceptance scenarios, never the intention behind it, and record the same
evidence you would demand from a worker.

## 5. Verify ([verification.md](../verification.md))

Bug-fix proof = the reproduction that failed before now passes. The red run above is itself
evidence the regression test can fail. Run the steps as written, then record each `run` and
its observed output in `tasks[].evidence` — timestamps from the system clock, never typed in
by hand.

## 6. Close (step 8)

Complete only when A1–A3 pass together. Revisit the open gap (other NOT NULL columns from
optional fields): either a fact ("checked, none") or a new task — do not leave it silently
absorbed ([decomposition.md](../decomposition.md) replanning).
