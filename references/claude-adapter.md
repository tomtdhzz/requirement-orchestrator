# Claude Adapter

Use Claude Code's available Agent/subagent capabilities for bounded work. Treat the installed tool surface, hooks, and project instructions as authoritative.

The skill modes `analyze`, `diagnose`, `execute`, and `challenge` are semantic modes, not Claude Code's native Plan Mode. Do not call `EnterPlanMode` merely because this skill is analyzing or diagnosing. If native Plan Mode is already active, preserve the user's read/write constraints and use workers only when this skill's delegation gates are met.

## Dispatch

- Give each worker the common contract from `agent-contract.md`.
- Use separate workers only for independently verifiable tasks.
- Pass required files and decisions explicitly; do not rely on private context from the controlling conversation.
- Bring results back to the shared ledger in `review` state.
- Maintain the phased TODO progress surface with the native `todo` tool (`init` with `list: [{phase, items}]`, then `start`/`done`); advance it from ledger state, not from worker self-reports.

## Control

- The main Claude session remains controller unless a handoff snapshot explicitly transfers control.
- Apply the same dependency, write-scope, contract-freeze, and verification gates used by Codex.
- Do not encode different task semantics merely because Claude uses different agent definitions or hooks.
- If worker behavior is supplied by project-local agent files, those files adapt dispatch mechanics; this skill remains the semantic source of truth.

If Claude lacks usable subagent capability, preserve the task graph and execute sequentially or hand control to Codex through the ledger snapshot.
