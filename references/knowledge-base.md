# Capability Knowledge Base (optional; consult first when configured)

**Read this file only when a capability knowledge base is already configured.** It is an
optional integration, not a step of the control loop: with no base configured, prior art
comes from the repo and the installed skills ([context-grounding.md](context-grounding.md))
and nothing here applies. When one *is* configured, query it at the START of `analyze`,
before decomposition, instead of researching from scratch — then prefer reusing or
importing over building.

The reference implementation is
[skills-radar](https://github.com/tomtdhzz/skills-radar): a multi-source, deduped, ranked
catalog of Agent Skills sunk into an Obsidian vault. Set `$RADAR` to your clone path. Any
tool that maps a goal to a ranked capability shortlist works here; adapt the commands.

## What it answers
Given a goal (e.g. "build an AI video-generation product"), it returns the relevant
capability set plus a same-domain comparison so you can pick, not search.

## How to consult
```bash
# 1. Query capabilities for the goal (Chinese or English). Prints matched domains
#    + a comparison table (sources / installs / stars / score / install command).
python3 $RADAR/kb.py query "<goal or domain keywords>"

# 2. (optional) Browse in Obsidian for human review:
#    $RADAR/vault/index.md  ->  domains/<domain>.md (MOC)  ->  compare/<domain> compare.md
```

## How to use the result in the control loop (step 1)
1. Run the query; read the matched domains and the comparison table.
2. Present the candidate capability set + comparison to the user; let them (or the
   controller, per taste) pick which existing skills to reuse/import.
3. Record the chosen capabilities into the ledger `analysis.facts` and `decisions`
   (id, source, why chosen). Prefer reuse/import over building from scratch:
   `npx skills add <id> -g -y`  ·  `npm i <pkg>`  ·  `sr install <id>`
   (or `python3 $RADAR/crawl.py pull <id>`).
4. Decompose the task AROUND the chosen capabilities.

## Keep it fresh
```bash
python3 $RADAR/crawl.py crawl && python3 $RADAR/kb.py export
```

## Semantic layer (optional, additive)
The deterministic query is rule + domain based. For fuzzy matching, layer local embeddings
over `$RADAR/vault` (Smart Connections or obsidian-notes-rag MCP); `kb.semantic_hits()` is
the wiring point. Do not require it — deterministic works offline.

## Boundary
KB output is a recommendation to verify, not an automatic install. Do not assert a
capability fits beyond what the query/comparison shows; confirm before depending on it.
