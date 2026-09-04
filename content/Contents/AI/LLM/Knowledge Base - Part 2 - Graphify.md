---
aliases:
  - Knowledge Base Graphify
  - Obsidian knowledge graph
tags:
  - AI
  - LLM
  - Local_LLM
  - Tools
---

This is Part 2 of building a local knowledge base on top of my Obsidian vault. [[Knowledge Base - Part 1 - Setup|Part 1]] covered installing Obsidian and Hermes and wiring Hermes up to a GLM coding plan key. This part puts that to work: turning the vault into a persistent, queryable knowledge graph with **graphify**.

### What is graphify?
graphify turns any folder of files — notes, docs, papers, code, images — into a knowledge graph with community detection. It produces three things, written to a `graphify-out/` folder in the target directory:

- an interactive HTML visualization of the graph
- a `graph.json` file that is GraphRAG-ready
- a plain-language `GRAPH_REPORT.md` summarizing the clusters it found

In Hermes it ships as the `/graphify` skill, bundled at `~/.hermes/skills/graphify/`. The first time the skill runs it installs itself (the `graphifyy` Python package via `uv tool` or `pip`) — there is nothing to install by hand. To pre-install manually:
```bash
uv tool install --upgrade graphifyy
```

### Prepare the vault
Two minutes of housekeeping before the first run:

1. **Back up the vault.** If it is a git repo, commit first. graphify only reads the vault and writes to `graphify-out/`, but backups are cheap.
2. **Plan for the output folder.** After the first run, add `graphify-out/` to the vault's `.gitignore` (if the vault is a git repo) and to Obsidian's excluded files under Settings → Files & Links → Excluded files, so Obsidian does not index the generated artifacts as if they were notes.

One nice property: an Obsidian vault is already plain Markdown with `[[wikilinks]]`. graphify does its own semantic extraction, so the existing link structure is a bonus, not a requirement.

### Semantic extraction (optional but recommended)
Semantic extraction — the INFERRED edges between concepts in different notes — can run two ways:

- **With a Gemini key:** set `GEMINI_API_KEY` (or `GOOGLE_API_KEY`) in `~/.hermes/.env` and install the Gemini extra (`pip install 'graphifyy[gemini]'`). Default model is `gemini-3-flash-preview`, overridable with `GRAPHIFY_GEMINI_MODEL`. This is the fast, cheap path for big vaults.
- **Without one:** graphify falls back to the host agent doing extraction with subagents — in our setup that means the GLM model from Part 1 burning coding-plan credits. It works, it is just slower and uses your quota.

It ignores `ANTHROPIC_API_KEY` and `OPENAI_API_KEY` either way, and pure code-structure mapping needs no key at all — but that part only matters for code, not notes.

### Build the graph
Open Hermes inside the vault and invoke the skill:
```bash
cd ~/Documents/obsidian-vault
hermes
```
Then in the chat:
```
/graphify                          # full pipeline on the current directory
/graphify <path>                   # or on a specific subfolder
```
The first run takes a while on a large vault. When it finishes, take a look at the outputs:

- `graphify-out/index.html` — open in a browser, this is the visualization
- `graphify-out/GRAPH_REPORT.md` — opens like any other note in Obsidian
- `graphify-out/graph.json` — the machine-readable graph

### Useful flags
```
/graphify <path> --mode deep      # thorough extraction, richer INFERRED edges
/graphify <path> --update         # incremental — only re-extract new/changed files
/graphify <path> --svg            # also export graph.svg (embeds in Notion/GitHub)
/graphify <path> --graphml        # export graph.graphml (Gephi, yEd)
/graphify <path> --neo4j          # generate cypher.txt for Neo4j
/graphify <path> --wiki           # build an agent-crawlable wiki
/graphify <path> --watch          # auto-rebuild on file changes (no LLM needed)
/graphify --help                  # print the full usage block
```
For an Obsidian vault, `--wiki` is worth a try — it generates crawlable wiki pages that read naturally inside Obsidian.

### Keep it fresh
Once `graphify-out/graph.json` exists, questions go straight to the graph — no rebuild needed:
```
/graphify query "What do my notes say about data ingestion patterns?"
/graphify query "trace how Snowflake connects to dbt" --dfs
/graphify path "Kafka" "Snowflake"          # shortest path between two concepts
/graphify explain "GraphRAG"                # plain-language explanation of a node
/graphify add <url>                          # fetch a URL into the corpus and update the graph
```
Extraction results are cached, so re-running with `--update` after a writing session only processes the notes that changed. If a chunk fails mid-extraction, just re-run — the cache means only the failed files are retried.

### Tips
- Very large corpora (over ~2M words or ~500 files) trigger a prompt to narrow to a subfolder — pick one to keep extraction fast and cheap. For a big vault, graph one folder at a time (`Contents/`, `Work/`, etc.).
- Run `--update` after writing sessions and keep `--mode deep` for occasional full rebuilds.
- The graph reflects the vault: naming concepts consistently in notes produces much cleaner clusters.

That is the whole pipeline: a local Obsidian vault, a Hermes agent running on the GLM coding plan, and a persistent knowledge graph over everything you have written — no data ever leaves your machine except the model calls.
