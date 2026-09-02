# Example — Bug flow (diagnose → execute)

Illustrative end-to-end run of a reported failure. Shows reproduction-before-root-cause,
single root cause per repair path, and evidence-gated closure.

**Report:** "Checkout intermittently returns 500."

## 1. Reproduce before claiming a cause (diagnose)

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

## 3. Spec the fix (one root cause, one repair path)

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
```

Single root cause + single repair path ⇒ no delegation; the controller uses the control
loop directly. Fix source (make `coupon_id` nullable or omit the field when absent — choose
per schema intent), not the symptom (don't swallow the 500).

## 4. Verify ([verification.md](../verification.md))

Bug-fix proof = the reproduction that failed before now passes. Run A1 (was 1/20 500s →
0/200), A2, and the new A3 regression. Record in `tasks[].evidence`.

## 5. Close (step 8)

Complete only when A1–A3 pass together. Revisit the open gap (other NOT NULL columns from
optional fields): either a fact ("checked, none") or a new task — do not leave it silently
absorbed ([decomposition.md](../decomposition.md) replanning).
