# Subagent Contract

Dispatch only when every required field is concrete:

```yaml
id: T1
goal: "One observable outcome"
background: "Why this work exists"
facts: []
dependencies: []
read_scope: []
write_scope: []
forbidden_changes: []
deliverables: []
acceptance: []
verification: []
```

Provide only the relevant ledger facts, contracts, project instructions, and source locations. Do not send the full conversation or unrelated task branches.

`verification:` entries are runnable, not intentions. Each is a command plus the observable outcome it must produce; a step that must fail first names the exact failure text, so the worker cannot mistake a different failure for the expected one:

```yaml
verification:
  - run: "go test ./internal/auth/..."
    expect: "ok, 0 failures"
  - run: "./bin/tool sync --dry-run"
    expect: 'FAIL — "unknown flag: --dry-run"'
```

An expectation a third party could not observe — "works correctly", "no regressions" — is not verification. Replace it with the command and output that would prove it, or drop the step.

Before the first dispatch that changes code, record the baseline result of the project's enforcement command — formatter / linter / tests, as bound during grounding — in the ledger; a later failure is otherwise unattributable (see [verification.md](verification.md)).

## Worker obligations

The worker must:

1. read the assigned contract and applicable project instructions;
2. stay within the authorized write scope;
3. report discoveries that invalidate dependencies or contracts;
4. run the specified verification where possible;
5. return changed artifacts, verification evidence, remaining risks, and a status of `review`, `blocked`, or `failed`.

The worker must not broaden product scope, change a frozen shared contract, or mark itself completed.

## Review dispatch

A review is worth only its independence, so the request must not pre-judge its verdict. The rule and its "do instead" live with the review standards themselves, in [verification.md](verification.md) — read it when dispatching a review, not only before the first task dispatch.

## Controller review

Check observable artifacts rather than trusting a success summary. Verify:

- the requested behavior and acceptance criteria;
- the authorized write boundary;
- dependency and shared-contract compatibility;
- relevant tests or commands;
- unresolved risks and unintended changes.

The controller may make a small correction only when it does not change behavior, contract, or scope. Otherwise return the task to its worker with specific findings and re-run verification — a second pass verdicts the recorded findings rather than restarting the review.

This list is the short form. [verification.md](verification.md) holds the authoritative review checklist, the re-review protocol, and the evidence standards; keep it as the single source rather than growing a second one here.

