# Codex Adapter

Use Codex's available collaboration tools for bounded subagent work. Treat the actual tool list and project instructions as authoritative.

The skill modes `analyze`, `diagnose`, `execute`, and `challenge` describe the requested outcome and authorization boundary. They do not automatically select a product UI mode or authorize file changes.

## Dispatch

- Dispatch independent tasks separately; keep dependent work ordered.
- Give each subagent the contract from `agent-contract.md` and only its relevant context.
- Do not copy the entire controller conversation when a clean task context is sufficient.
- Keep the controller productive while workers run; collect each result into ledger `review` state.
- Maintain the phased TODO progress surface with Codex's native plan/update-plan mechanism; advance it from ledger state, not from worker self-reports.

## Control

- The main Codex session remains controller unless a handoff snapshot explicitly transfers control.
- Use follow-up work on the same worker for review findings when the runtime supports it.
- Do not assume agents have isolated filesystems. Apply the parallel write gate before dispatching writers.
- A tool-reported completion is evidence to inspect, not final acceptance.

If Codex lacks subagent capability in the active environment, preserve the task graph and execute sequentially in the main session or offer a controlled handoff to Claude.
