# Safe Bulk Mutation

Applies when `execute` changes many existing artifacts — documents, configs, generated files, mass code edits, or records in an external system. Here the dominant failure is silent data loss or partial application, not logic error. The discipline below is mandatory whenever a step rewrites, deletes, or fans a change across more than one existing artifact.

## Non-destructive by default

- Prefer a surgical insert or patch over clear-and-rewrite. Rewriting a whole artifact destroys anything the task did not author and cannot regenerate — human edits, comments, screenshots, adjacent configuration.
- Before any rewrite, identify what in the target was not produced by this task. If such content exists and is not reconstructable from a source of truth, do not rewrite; insert or patch in place.
- Treat an artifact that may have been edited by others as authoritative for the parts you did not write.

## Idempotent and rerun-safe

- Mark inserted content with a stable, detectable marker so a re-run detects it and skips or updates in place instead of duplicating.
- Every mutation step must be safe to run twice. Detect-then-act; never blind-append.

## Pilot, then batch

- Apply to one representative target first. Read it back and confirm placement and content before fanning out.
- Only after the pilot verifies do you run the batch. Keep external writes serial or within the target's proven-safe concurrency — silent rate-limit failures corrupt a batch invisibly and leave a partially written result that looks complete.

## Verify by delta and independent read-back

- Do not trust the tool's own success count. Re-read each mutated target from the system of record and check it independently.
- Prefer a structural delta that proves nothing was lost: the observed change in size, block count, or line count equals exactly what was added — no more. An unexpected delta means something was deleted.
- Record per-target pass/fail in the ledger evidence. A failed or unverifiable target returns to `in_progress`, never `completed`.
