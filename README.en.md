# Requirement Orchestrator

**English** · [中文](README.md)

**A delegation-and-acceptance control layer on top of however you already work.** If the repo runs a spec framework, use its spec; if it has none, stand up the thinnest spec that can be accepted. It owns exactly two questions — **what must be frozen before work fans out**, and **what counts as done**.

Built for one person. When one person drives several agents at once, the bottleneck is not throughput but **acceptance**.

## What it is there to stop

> You dispatch two agents to edit **different files** in the same Go package. The write scopes do not overlap, so it looks perfectly parallel.
> Both agents' `go test` runs come back red — because the other's half-written file is in the compile unit. It takes half an hour to realize the problem is not in your code.

> Another agent reports "done, tests pass". You believe it. Three days later the change is nowhere in the diff — it had run a suite that was already green before it started.

What these share: **when the failure happened you had impressions, not criteria.** Criteria are what this skill supplies:

- Whether two tasks may run at once is decided by **whether they share a compile/test target** — not by "they touch different files". The latter is the common wrong answer, and it is exactly what produced the first story.
- Whether work is done is decided on the **actual `base_commit..HEAD` diff** — not on the worker's report. The second story is the price of trusting the report.

## 30 seconds to start

```bash
npx skills add tomtdhzz/requirement-orchestrator -g -y
```

Then say this to your agent — no spec needed, no changes to your repo:

> Use requirement-orchestrator to analyze this request: add rate limiting to the orders API.

**It will produce requirements and acceptance scenarios, then a task split with a parallel-safety verdict, and stop for your confirmation** — `analyze` is read-only, and changing code needs a separate authorization from you. That stop is the design, not a stall.

## How it differs

| You ask | A skills collection like superpowers | A host's built-in orchestrator mode | This skill |
|---|---|---|---|
| What must exist before I start? | install a skills set, brainstorm a plan doc first | nothing (but also no spec or acceptance concept) | **nothing** — no repo changes, no spec required |
| Who decides it is done? | evidence before claims, self-assessed by the doer | the model's own account | **the controller, judging the diff; a worker may only report `review`** |
| Can two tasks run at once? | heuristic: different test files / subsystems | not decided | **criterion: do they share a compile/test target — and the scan must be recorded** |
| Does it survive a host change? | Claude-centric | bound to that host | **plain Markdown; Codex and Claude interchangeably** |

Same-layer spec and task frameworks (spec-kit, OpenSpec, BMAD, ccpm, task-master) each need an artifact before you can start: a spec, a current-truth layer, `uv` plus a renderer, a GitHub issue, an API key. This one needs none. **"Lightweight" is an adjective; "zero prerequisites" is a checkable fact.**

As for hosts (Claude Code / Codex / omp): they provide *what can be done*, not *what counts as done*. A host with an orchestrator mode gives you **routing, not a gate**. Meanwhile a host can do what this skill cannot — **block** an action (hooks, CI). The two compose: **capabilities get absorbed by hosts; criteria do not.**

## When not to use it

- **A one-off change you will just make yourself** — the value is in delegation and acceptance; with no delegation most of the machinery degrades.
- **You want a board, progress sync, or team collaboration** — explicit non-goals; use ccpm or your ticketing system.
- **You want help writing the spec** — that is spec-kit / OpenSpec territory; this only stands up a minimal acceptable spec when none exists.
- **You expect it to prevent an agent from doing damage** — it cannot. It makes violations detectable; actual prevention belongs in host-side hooks.

## What it actually produces

One spoken request goes in; what comes back is not plan prose but four checkable things:

1. **A spec thin enough to accept** — one requirement, one acceptance scenario, a frozen contract (if the repo already has a spec, that one is used; a second is never created)
2. **A parallel-safety verdict with its scan table** — one row per task pair sharing a file, compile/test target or interface; `"the scan is clean"` without those rows is not a scan that was run
3. **Verification steps that can be re-run** (the dispatch contract carries commands, not intentions)

   ```yaml
   verification:
     - run: "npx jest test/api.rateLimit.int.spec.ts"
       expect: "PASS — 429 body + Retry-After header present"
   ```

4. **Completion judged on the diff** — a worker saying "done" does not settle it; the controller reads `base_commit..HEAD`, re-runs the command above, checks the write scope held, and only then records `completed`

## Dogfooding: we develop it with itself

All 51 commits in this cycle went through the process (stand up a spec, dispatch read-only audits, judge on the diff, keep the ledger). The defects it caught were its own:

- **7 cases of "the rule was written but unreachable where it applies"** — two of them unreachable the day they landed (a newly added confirmation gate living in a file that step never loads). Only a systematic reachability check finds this class.
- **4 cases of "the edit reported success and the content was wrong"** — including one that silently deleted a checklist line. All caught by independently reading the diff, never by the tool's success receipt.
- **7 candidate features judged "not this project's to own"** — cheap, frequent and writable, but belonging to spec production or task scheduling.

These are not marketing numbers; they are line-by-line checkable in this repo's `CHANGELOG.md` and git history.

## Three pillars

- **Zero prerequisites** — no spec, nothing to install, no task store, no account.
- **Input-shape agnostic** — a feature, a change, a bug, "take a look at this code" run through one control loop. A bug is just one input shape, distinguished only by one extra step: reproduce before claiming a root cause.
- **Authority separated from evidence** — a worker may only submit `review`; only the controlling agent records `completed`, on evidence.

## Roadmap

**In progress**

- **A retirement mechanism for temporary work.** Mitigation on the bug side, a hardcoded value on the feature side, a deferred finding on the review side — today each can be dressed up as a permanent solution and pass acceptance, and once recorded `completed` it is never reclaimed. The criterion: temporary work must be **deliberate, recorded, and carry its own retirement condition**.
- **A finer worker report contract**: distinguish "done with concerns" and "missing context" from a bare `review`, so the controller can act on a status instead of reading prose.

**Next**

- **A `Freeze Before Fanout` concept document**, English and Chinese. The three freezes — contract, scope (bounded by the compile/test target), and facts with their provenance — are the one thing here nobody else has named.
- **Composition proof**: run a full cycle in a real repo that already has spec-kit or OpenSpec, and record what "their spec, our gates" looks like end to end.
- **Push unenforceable rules down to the host**: ship optional hook examples (echo the diff after an edit, block a mixed-concern commit) rather than building a runtime into the skill.

**Never**

A cloud backend or accounts · a task store, board or dashboard · generating the spec itself · task-ordering algorithms · a runtime bound to one host. These are not "not yet" — each was judged out of this project's domain, with the reasoning recorded.

## Going deeper

The specification is [`SKILL.md`](SKILL.md) (90 lines, always resident); the detail is in [`references/`](references) (15 files, loaded on demand, ~950 lines). On any conflict with this README, `SKILL.md` wins.

- Shape the spec and freeze contracts → `spec-driven.md`; ground before decomposing → `context-grounding.md`
- Decomposition, the parallel gate, replanning → `decomposition.md`; acceptance evidence, finding routing, re-review convergence → `verification.md`
- Subagent contract → `agent-contract.md`; ledger and resuming after an interruption → `ledger.md`; read-back for bulk and single-line edits → `mutation.md`
- Stress-testing an existing requirement/design/implementation → `challenge.md`; platform differences → `*-adapter.md`

## Requirements & dependencies

- A pure **methodology skill (Markdown only)**: no runtime, no `pip`/`npm` dependencies, nothing to install.
- An **agent host** that can read `SKILL.md`: Claude Code (omp), Codex, or any agent you point at it.
- Optional integration: [skills-radar](https://github.com/tomtdhzz/skills-radar) as a starting capability base (see `references/knowledge-base.md`).

## Limitations & non-goals

- **No enforcement.** A pure prompt artifact: its rules give you an **observable check, not a block**. They let a violation be caught afterwards; they cannot stop one. Real enforcement lives in tooling.
- Consequently [`references/verification.md`](references/verification.md)'s evidence standards are the only part that converts good intentions into something checkable.
- **Resident cost is real.** `SKILL.md` enters context every session — hence every rule states a retirement condition (see `Retiring a rule` in [CONTRIBUTING.md](CONTRIBUTING.md)).
- **The parallel gate is a criterion, not a lock.** It tells you whether two tasks may run at once; it does not isolate them.
- **The ledger is a file, not a database.** Nothing arbitrates when it is stale — on recovery, treat every `in_progress` task as unverified.
- **Non-goals**: no task store, board or dashboard; no hosted state; no spec generation of its own; no strongly consistent cross-platform state.

## License & contributing

- License: [MIT](LICENSE)
- Contributing: see [CONTRIBUTING.md](CONTRIBUTING.md). Rules are graded with RFC 2119, and each must state its gap, consequence class, cost location, and **retirement condition** — favoring fewer, sharper rules over a growing checklist.
- Changes: [CHANGELOG.md](CHANGELOG.md).
