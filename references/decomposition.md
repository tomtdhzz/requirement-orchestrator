# Decomposition and Scheduling

## Choose one primary axis

Choose the axis that best matches the change's cohesion:

| Signal | Primary axis |
| --- | --- |
| A concrete defect or regression | Bug root cause and repair path |
| Independent user-visible capabilities | Feature |
| Services can be delivered and verified independently | Service |
| Business rules, state, or consistency span components | Domain |

When domain and service boundaries both matter, use domain for business responsibility and acceptance, and service for implementation location. Build one task tree; record the other dimension as labels and read/write scopes.

## Bug and failure triage

Treat multiple error messages as symptoms until evidence shows independent causes. Prefer one task for one root cause and repair path.

For build or deployment failures involving missing symbols, configuration, generated artifacts, or unexpectedly absent capabilities, check repository state before assuming a code defect:

- current branch, HEAD, and working-tree state;
- divergence from the intended base branch;
- merge base and whether required commits are ancestors of HEAD;
- relevant diffs between the base and current branch;
- signs that a merge, cherry-pick, or rewrite removed an existing capability.

Repository state is a diagnostic lens, not a fifth decomposition axis. Once the cause is established, keep the primary axis as bug root cause and repair path.

Use at most two decomposition levels. Stop when each task has one goal, explicit boundaries and dependencies, a self-contained context, and independent acceptance.

## Dependencies are explicit

Record `depends_on`; never infer execution order from parent/child position. Freeze shared API, schema, state semantics, identifiers, error contracts, and process-global registrations or singletons before dependent implementation begins.

## Parallel gate

Writing tasks may run in parallel only when all are true:

- dependencies are resolved;
- write scopes do not overlap;
- shared contracts are frozen;
- validation can run independently.

Different files, services, repositories, or AI platforms do not by themselves prove independence.

The unit of independent validation is the toolchain's compile/test target, not the file. Two tasks that edit different files in the same package/module/test target cannot validate independently — one's half-written file breaks the other's build or test run. Split parallel writing at that boundary (e.g. one agent per Go package or per service, not per file inside a shared package); if work must share a compile unit, serialize it or give it to one agent.

## Replanning

If a worker discovers an out-of-scope change, it reports the affected boundary without modifying it. The controller decides whether to expand authority, create an upstream task, change ordering, or revise the decomposition.

On failure, classify the cause before retrying: missing context, invalid contract, excessive task size, environmental blocker, or implementation error. Retry the same approach at most once; a repeated failure requires a changed plan, smaller task, controller takeover, or an explicit blocker.
