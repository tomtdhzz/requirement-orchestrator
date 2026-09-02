# Optional Trellis Adapter

Trellis is an escalation path, not a prerequisite and not a second controller.

## Suggest escalation when

- work must continue across sessions;
- there are at least three independently deliverable child tasks;
- multiple agents require multi-turn or asynchronous coordination;
- durable, auditable decisions and messages are needed;
- persistent parent/child tasks or independent checking materially improve control.

User confirmation is required before changing the workflow to use Trellis.

## Mapping

- Map the source requirement and cross-child acceptance to a parent task.
- Map independently deliverable work to child tasks.
- Keep dependencies explicit in task artifacts; Trellis parent/child position is not scheduling order.
- Map relevant facts and project rules to PRD, design, implementation, research, and context artifacts without maintaining a second source of truth.

## Challenge mode

Trellis provides the task and collaboration runtime; an independent Challenge or Check agent performs the critique. Ask it to examine requirement omissions, domain boundaries, cross-task contracts, failure paths, compatibility, deployment risk, and acceptance blind spots without redefining the confirmed product goal.

Route findings back to the controller. Rework only the affected requirement, decomposition, design, or implementation branch.

Use a Trellis channel only for multi-turn cross-platform conversation, asynchronous peer work, or durable message history. A one-time Codex/Claude handoff needs only the ledger snapshot.

