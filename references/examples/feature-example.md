# Example — Feature flow (analyze → authorized → execute)

Illustrative end-to-end run of a feature through the control loop. Abridged; shows how the
spec model, grounding, decomposition, contracts, dispatch, and verification connect.

**Request:** "Add rate limiting to our public REST API."

## 1. Ground + spec (analyze, read-only)

Context grounding (read-only) recorded as facts: API is an Express app in `src/api/`;
existing middleware chain in `src/api/mw/`; Redis already used for sessions (`src/redis.ts`);
no current limiter; two handlers already read `req.userId`. Enforcement command bound here:
`npm run lint && npx tsc --noEmit && npx jest` — one TypeScript project, one jest target.

Spec ([spec-driven.md](../spec-driven.md)):
- **Requirements:** R1 authenticated callers limited per-user; R2 unauthenticated limited
  per-IP; R3 over-limit returns 429 + `Retry-After`; R4 limits configurable without deploy.
  Non-goals: no per-endpoint tiers (v2), no distributed fairness guarantees.
- **Acceptance scenarios:** A1 *Given* a user at the limit *When* another request *Then* 429
  + `Retry-After`. A2 per-IP path for anonymous. A3 config change takes effect without
  redeploy. A4 under-limit traffic unaffected (latency budget stated).
- **Contracts (frozen):** `RateLimiter.check(key): {allowed, retryAfter}`; Redis key scheme
  `rl:{scope}:{id}`; 429 body shape; config schema `{windowSec, max}`.

## 2. Authorization gate

The spec and the frozen contracts are surfaced for confirmation. `analyze` produced them and
stopped: nothing in the request ("add rate limiting") authorizes edits. The user confirms the
spec and says to implement it — that separate instruction is what opens `execute`.

Then, before the first dispatch that changes code, the baseline is recorded in the ledger:
`npm run lint && npx tsc --noEmit && npx jest` → clean (`142 passed`). A later failure is now
attributable.

## 3. Decompose (axis: feature; two levels)

```yaml
tasks:
  - id: T1                 # the frozen contract lives here; built by controller first
    goal: "RateLimiter interface + config schema + 429 shape"
    depends_on: []
    write_scope: [src/api/rateLimiter/types.ts, src/api/config/limits.ts]
    acceptance: ["A3 config shape loads and is read at request time"]
    verification:
      - run: "npx tsc --noEmit"
        expect: "no errors"
      - run: "npx jest test/config.limits.spec.ts"
        expect: "PASS — loads {windowSec,max}, rejects negative max"
    status: pending
  - id: T2
    goal: "Redis-backed limiter implementing RateLimiter.check"
    depends_on: [T1]
    write_scope: [src/api/rateLimiter/redisLimiter.ts]
    acceptance: ["A1/A2 unit-level behavior against the frozen contract"]
    verification:
      - run: "npx jest test/rateLimiter.spec.ts"
        expect: "PASS — 6 tests, incl. boundary request at max"
    status: pending
  - id: T3
    goal: "Express middleware wiring per-user/per-IP + 429 + Retry-After"
    depends_on: [T1, T2]
    write_scope: [src/api/mw/rateLimit.ts]
    acceptance: ["A1/A2/A3 end to end; A4 latency budget"]
    verification:
      - run: "npx jest test/api.rateLimit.int.spec.ts"
        expect: "PASS — 429 body + Retry-After header present"
    status: pending
integration:
  cross_task_contracts: ["RateLimiter.check(); rl:{scope}:{id}; 429 body; {windowSec,max}"]
  next_action: "Build T1, freeze, then T2, then T3 — one writer at a time in this target"
```

Parallel gate ([decomposition.md](../decomposition.md)) — the naive reading fails here. T2
and T3 depend only on the frozen T1 contract and write **disjoint files**, which looks
parallel-safe. It is not: both files compile and test under the same TypeScript project and
the same jest target, and the unit of independent validation is the toolchain's compile/test
target, not the file. T2 half-written breaks `tsc` for T3's test run, so neither result is
attributable. Resolution: `depends_on: [T1, T2]` on T3 and one writer at a time. Genuine
parallelism here would require a target boundary that does not exist in this repo (e.g. a
separate package for the limiter).

## 4. Dispatch (T1 → T2 → T3, serialized)

Each worker gets the contract from [agent-contract.md](../agent-contract.md): goal, the
frozen `RateLimiter`/key/config facts, its write scope, forbidden changes (don't touch the
other tasks' files or the contract), its acceptance scenarios, and its `run`/`expect`
verification steps — commands, not intentions.

Phased TODO mirrors the groups: `契约` (T1) / `实现` (T2, T3) / `全量验收`, advanced only
when the controller records `completed`.

## 5. Review + verify ([verification.md](../verification.md))

Workers return `review`. Controller checks artifacts, not summaries: re-runs each task's
`run`/`expect` steps, confirms A4 latency, confirms write boundaries held, confirms the
declared status agrees with the report body, records evidence in `tasks[].evidence`; only
then marks `completed`. A returned task's second pass verdicts the recorded findings rather
than restarting the review.

## 6. Integrate + close (step 8)

End-to-end: exercise A1–A4 against the running API together; record in
`integration.final_verification`. Complete only when every acceptance scenario passes — not
when T1–T3 individually report done. Qualifying project lessons are appended to
`.ai-work/lessons.md`.

## What a challenge pass would flag ([challenge.md](../challenge.md))
Robustness: Redis down → fail-open or fail-closed? (unspecified → finding). Concurrency:
two simultaneous requests at the boundary (race on counter → needs atomic INCR). These
become findings with the smallest fix, for the controller to fold in.
