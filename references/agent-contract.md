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

## Worker obligations

The worker must:

1. read the assigned contract and applicable project instructions;
2. stay within the authorized write scope;
3. report discoveries that invalidate dependencies or contracts;
4. run the specified verification where possible;
5. return changed artifacts, verification evidence, remaining risks, and a status of `review`, `blocked`, or `failed`.

The worker must not broaden product scope, change a frozen shared contract, or mark itself completed.

## Controller review

Check observable artifacts rather than trusting a success summary. Verify:

- the requested behavior and acceptance criteria;
- the authorized write boundary;
- dependency and shared-contract compatibility;
- relevant tests or commands;
- unresolved risks and unintended changes.

The controller may make a small correction only when it does not change behavior, contract, or scope. Otherwise return the task to its worker with specific findings and re-run verification.

