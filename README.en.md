# Requirement Orchestrator

> **Make an agent's "done" a checkable fact.**
> Zero install · no spec required · parallel safety decided by the **compile/test target**, completion by the **`base_commit..HEAD` diff**.

**English** · [中文](README.md) · [MIT](LICENSE) · Markdown only, zero dependencies

## TL;DR

A **delegation-and-acceptance control layer** for AI coding agents. It answers two questions: **what must be frozen before work fans out**, and **what counts as done**.

- Zero prerequisites: nothing to install, no spec required, no repo restructuring.
- Criteria, not impressions: parallel safety is decided by the **compile/test target**; completion by the **`base_commit..HEAD` diff**.
- Built for one person. With one human and several agents, the bottleneck is acceptance, not throughput.

## Features

**What it solves**

| Situation | Without it | With it |
|---|---|---|
| Two agents edit different files in one package | Both `go test` runs go red; half an hour to locate why | Parallel gate decides on the compile/test target, and the scan must be recorded |
| A worker reports "done, tests pass" | You trust the report; three days later the change is not in the diff | The controller reads `base_commit..HEAD`, re-runs the verification, then records `completed` |
| The request is one spoken sentence | Work starts, acceptance criteria surface too late | A minimal spec is stood up: one requirement, one acceptance scenario, a frozen contract |
| Resuming after an interruption | `in_progress` is treated as "half done" and pushed forward | Every `in_progress` task is unverified until its artifacts are re-checked |

**Core mechanics**

- One controlling agent owns the ledger, dispatch, review and integration; a worker may only submit `review`.
- Contracts freeze before fan-out, marked by a greppable `<frozen-after-approval>` boundary.
- Dispatch contracts carry `run` / `expect` commands, not intentions; an empty `verification` is an incomplete contract.
- Re-review verdicts only the recorded findings, capped at five rounds, adjudicated only at the cap.
- Dismissals need evidence too: `false` carries what disproves it, `unverified` carries what must still be checked.

**Not in scope**

- Task stores, boards, dashboards, progress sync.
- Generating the spec itself (if the repo has one, it is used; a second is never created).
- Task-ordering algorithms, cloud backends, a runtime bound to one host.

**Limitations**

- **No enforcement.** Rules are observable checks, not blocks. Real prevention belongs in host hooks or CI.
- The parallel gate is a criterion, not a lock; working-tree isolation is your git habit.
- The ledger is a file, not a database — nothing arbitrates when it goes stale.

## Quick start

```bash
npx skills add tomtdhzz/requirement-orchestrator -g -y
```

```text
Use requirement-orchestrator to analyze this request: add rate limiting to the orders API.
```

It returns the spec, acceptance scenarios, task split and parallel verdict, then **stops for authorization** — `analyze` is read-only; changing code takes a separate instruction.

## Configuration

**Semantic modes**

| Mode | Purpose | Edits code |
|---|---|---|
| `analyze` | Investigate and produce an execution blueprint (default, when nothing has failed) | No |
| `diagnose` | Reproduce and explain a failure (**any request about an existing failure routes here**) | No |
| `execute` | Dispatch, review, integrate | Yes, with separate authorization |
| `challenge` | Stress-test a requirement / design / implementation (code and PR review land here) | No |

**Where state lands**

| Path | Contents | Committed |
|---|---|---|
| `docs/prd/`, `docs/tech-design/` | Spec and technical design | Yes |
| `.ai-work/ledger.md` | Task state machine, evidence, decisions | No (gitignored) |
| `.ai-work/plan.md`, `lessons.md` | Phased plan, project lessons | No |

**Requirements**

| Item | Requirement |
|---|---|
| Runtime | None |
| Host | Any agent that can read `SKILL.md` (Claude Code / omp / Codex) |
| Optional | [skills-radar](https://github.com/tomtdhzz/skills-radar) as a starting capability base |

The specification is [`SKILL.md`](SKILL.md) (90 lines, always resident); detail lives in [`references/`](references) (15 files, loaded on demand). On conflict, `SKILL.md` wins.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). A rule change states four things:

| Item | Meaning |
|---|---|
| Gap | What goes wrong without it, backed by a real run or a clear failure mode |
| Consequence class | `irreversible` / `one wasted round` / `noise` |
| Cost location | Resident context / agent round-trip / human attention |
| Retirement condition | The signal that means delete it |

- A rule must be reachable in one hop from the control-loop step it governs, or it does not exist.
- A `noise`-class rule that costs resident context is rejected.
- `CHANGELOG.md` records only what a user would notice; internal wording fixes live in commit bodies.
