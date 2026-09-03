# Deliverable Artifacts Layout

Separate **publishable documentation** from **internal orchestration artifacts**. The first
is written for a reader of the project and ships with it; the second is process state for
controlling the work and must not pollute a published repo.

```
docs/                 # publishable — ships in the repo
├── prd/              # product requirements (spec) — spec-driven.md
└── tech-design/      # technical design (the "how") — tech-design.md

.ai-work/             # internal — orchestration state, NOT published
├── ledger.md         # requirement ledger (machine-state YAML) — ledger.md
├── plan.md           # execution plan mirroring the phased TODO / Progress surface
└── lessons.md        # experience log — experience.md
```

## Rules

- **Publishable vs internal — the key split.** `docs/` holds reader-facing documentation
  (PRD, technical design) that belongs in the repository. The ledger, plan, and lessons are
  working state, not project docs — keep them under `.ai-work/`, and for a publishable
  project add `.ai-work/` to `.gitignore`. Do not put the ledger/plan under `docs/`.
- **One subdirectory per doc type** under `docs/`; do not collapse PRD and tech design into
  one file; each is reviewed and evolves independently.
- **Prefer an existing docs location.** If the project already has a docs tree, map into it;
  otherwise create `docs/<type>/` at the project root; confirm the root if ambiguous.
- **Ledger format unchanged, home is internal.** The ledger's YAML still follows
  [ledger.md](ledger.md); its path is `.ai-work/ledger.md` (or `.trellis/tasks/` in an
  adopted Trellis flow) — never a published `docs/` path.
- **Plan mirrors the TODO.** The plan doc is the readable view of the phased TODO
  (Progress surface in SKILL.md); the native task list stays the live driver. One plan item
  ↔ one ledger task/verification step.
- **Cross-reference by relative path** and update links on any move.
- **Self-review the rendered output before handoff.** A written file is not a delivered
  artifact until it renders correctly for the reader. After generating or editing any
  Markdown deliverable, re-read it and check: heading/list/table structure is intact; every
  fenced block is closed and labeled; Mermaid and other diagram blocks actually parse
  (render them — e.g. `@mermaid-js/mermaid-cli` — do not eyeball syntax); links and relative
  paths resolve. Treat a parse/render failure as a defect to fix now. This is the
  `Document / diagram artifact` evidence standard in [verification.md](verification.md).

## Publishable project deliverables

When the deliverable is a standalone project or repository (not a change inside an existing
one), "done" includes being **publishable** — a stranger can clone, understand, build, and
run it. Ship, at the repo root:

- **README** — what it is and why; prerequisites; install/build; usage with runnable
  examples (CLI and/or API, incl. request/response); configuration and secrets handling;
  a short architecture overview; how to test; limitations/non-goals; license; and any legal
  disclaimer the domain requires.
- **LICENSE** — an explicit license (confirm the choice with the user if unclear).
- **.gitignore** — excludes build output, downloaded artifacts, `.ai-work/`, and anything
  secret; never commit credentials or tokens.

Keep publishable docs (`docs/prd`, `docs/tech-design`) in the repo — design docs are useful
to readers — but keep the internal `.ai-work/` artifacts out of it. Verify, don't assume:
actually run the README's commands yourself from the repo root, exactly as written —
including how the entrypoint is invoked (a bare `tool` name only works if it is on `PATH`;
otherwise show `./tool`, `go run ./cmd/...`, `npm start`, etc.). A source directory is not
an executable. In shell examples use copy-safe placeholders like `{BVID}` — never `<...>`,
which the shell treats as redirection and breaks a copy-paste run. And any embedded diagrams
must render.

## Where it enters the control loop

- Step 1 (`analyze`): create `docs/prd/` (spec) and, for non-trivial builds,
  `docs/tech-design/` (design).
- Step 3–4: keep the ledger, plan, and lessons under `.ai-work/` (gitignored when the
  project is publishable).
- Not every task needs all of these. A single-root-cause fix may keep only a ledger (or an
  in-conversation ledger per ledger.md). Scale the artifact set to the work.
