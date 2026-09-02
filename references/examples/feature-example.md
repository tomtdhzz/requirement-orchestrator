# Example — Feature flow (execute)

Illustrative end-to-end run of a feature through the control loop. Abridged; shows how the
spec model, grounding, decomposition, contracts, dispatch, and verification connect.

**Request:** "Add rate limiting to our public REST API."

## 1. Ground + spec (analyze → confirmed)

Context grounding (read-only) recorded as facts: API is an Express app in `src/api/`;
existing middleware chain in `src/api/mw/`; Redis already used for sessions (`src/redis.ts`);
no current limiter; two handlers already read `req.userId`.

Spec ([spec-driven.md](../spec-driven.md)):
- **Requirements:** R1 authenticated callers limited per-user; R2 unauthenticated limited
  per-IP; R3 over-limit returns 429 + `Retry-After`; R4 limits configurable without deploy.
  Non-goals: no per-endpoint tiers (v2), no distributed fairness guarantees.
- **Acceptance scenarios:** A1 *Given* a user at the limit *When* another request *Then* 429
  + `Retry-After`. A2 per-IP path for anonymous. A3 config change takes effect without
  redeploy. A4 under-limit traffic unaffected (latency budget stated).
- **Contracts (frozen):** `RateLimiter.check(key): {allowed, retryAfter}`; Redis key scheme
  `rl:{scope}:{id}`; 429 body shape; config schema `{windowSec, max}`.

## 2. Decompose (axis: feature; two levels)

```yaml
tasks:
  - id: T1                 # the frozen contract lives here; built by controller first
    goal: "RateLimiter interface + config schema + 429 shape"
    depends_on: []
    write_scope: [src/api/rateLimiter/types.ts, src/api/config/limits.ts]
    acceptance: ["types compile; config loads; matches frozen contract"]
  - id: T2
    goal: "Redis-backed limiter implementing RateLimiter.check"
    depends_on: [T1]
    write_scope: [src/api/rateLimiter/redisLimiter.ts]
    acceptance: ["A1/A2 unit tests green against contract"]
  - id: T3
    goal: "Express middleware wiring per-user/per-IP + 429 + Retry-After"
    depends_on: [T1]
    write_scope: [src/api/mw/rateLimit.ts]
    acceptance: ["A1/A2/A3 integration; A4 latency budget"]
integration:
  cross_task_contracts: ["RateLimiter.check(); rl:{scope}:{id}; 429 body; {windowSec,max}"]
  next_action: "Build T1, freeze, then fan T2+T3"
```

Parallel gate ([decomposition.md](../decomposition.md)): T2 and T3 both depend only on the
**frozen** T1 contract, write disjoint files, validate independently ⇒ parallel-safe. T1 is
the shared prerequisite, so the controller builds it inline first, then fans out.

## 3. Dispatch (T2, T3 in parallel)

Each worker gets the contract from [agent-contract.md](../agent-contract.md): goal, the
frozen `RateLimiter`/key/config facts, its write scope, forbidden changes (don't touch the
other's files or the contract), and its acceptance scenarios + how to verify.

## 4. Review + verify ([verification.md](../verification.md))

Workers return `review`. Controller checks artifacts, not summaries: runs A1/A2 unit +
integration; confirms A4 latency; confirms write boundaries held; records evidence in
`tasks[].evidence`; only then marks `completed`.

## 5. Integrate + close (step 8)

End-to-end: exercise A1–A4 against the running API together; record in
`integration.final_verification`. Complete only when every acceptance scenario passes — not
when T1–T3 individually report done.

## What a challenge pass would flag ([challenge.md](../challenge.md))
Robustness: Redis down → fail-open or fail-closed? (unspecified → finding). Concurrency:
two simultaneous requests at the boundary (race on counter → needs atomic INCR). These
become findings with the smallest fix, for the controller to fold in.
