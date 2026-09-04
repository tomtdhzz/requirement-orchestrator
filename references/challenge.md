# Challenge Mode

`challenge` stress-tests an existing requirement, decomposition, design, or **implementation**
for omissions and risks **without changing its confirmed product goal**. The deliverable is a
findings list with severity and evidence, not edits. Use it before committing to a plan, when
a plan smells too clean, or when the request is to review code that already exists — a review
asked for as "look at this PR" is a challenge over an implementation, and gets the same
severity-and-evidence discipline rather than free-form commentary.

## Method: quality-dimension dialectic

Advocate, in turn, for each quality objective as if it were the only thing that mattered,
then surface where they conflict. Deliberately introducing this tension exposes trade-offs a
single pass hides. Cover at least:

- **Correctness** — where could the spec be satisfied yet the behavior be wrong?
- **Completeness** — which requirement has no acceptance scenario? which path is unhandled?
- **Robustness** — failure, timeout, partial application, retry, concurrency, empty/malformed input.
- **Security / permissions** — authz, secrets, injection, least privilege, blast radius.
- **Performance / cost** — allocation, N+1, rate limits, unbounded growth.
- **Maintainability** — a third convention beside two existing; hidden coupling.
- **Operability** — how is it observed, rolled back, and verified in production?

## Omission checklist

- **Unstated non-goals** presumed in scope (or vice versa).
- **Missing acceptance:** a requirement no scenario would catch failing.
- **Unfrozen contract** shared by tasks marked parallel-safe.
- **Assumption posing as fact** in `analysis.facts` with no provenance.
- **Dependency inferred from task-tree position** rather than a real `depends_on`.
- **Verification that cannot fail** — "it works", no observable evidence.
- **Replanning trigger ignored** — a discovered out-of-scope change silently absorbed.

## Risk classes to rate

Missing context · invalid/again-unfrozen contract · oversized task · environmental blocker ·
silent data loss on mutation · irreversible/external action without read-back.

## Rules

- Produce findings as `{severity, where, evidence, smallest fix that unblocks}`. Order by
  severity; separate confirmed from suspected.
- Ground every finding; a challenge that asserts a risk without evidence is itself a
  fabrication. Mark suspected risks as suspected.
- Do not redesign. Propose the smallest change that removes the risk; the controller decides
  whether to fold it into the plan.
- Challenge does not change the confirmed product goal. If the goal itself looks wrong, say
  so as a finding and stop — do not substitute a different goal.
