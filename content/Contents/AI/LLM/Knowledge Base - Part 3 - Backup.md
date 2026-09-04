---
aliases:
  - Knowledge Base Backup
  - Obsidian GitHub backup
tags:
  - AI
  - LLM
  - Tools
---

This is Part 3 of building a local knowledge base on top of my Obsidian vault. [[Knowledge Base - Part 1 - Setup|Part 1]] covered the setup, [[Knowledge Base - Part 2 - Graphify|Part 2]] covered building the knowledge graph with graphify. This part is the boring but important one: backing the whole thing up on GitHub, so a dead laptop does not take years of notes — or the graph — with it.

### Why git?
An Obsidian vault is a folder of plain Markdown files, which is exactly what git was built for. GitHub gives us free private repositories, full version history of every note, and a restore path to any machine. The graph from Part 2 also lives in the vault folder (`graphify-out/`), so one repo can carry both.

### Step 1: Clean the vault first
Before the first push, decide what belongs in the repo. Create or edit `.gitignore` in the vault root:

```gitignore
.DS_Store
.obsidian/workspace.json
graphify-out/
```

- `.obsidian/workspace.json` changes every time you open Obsidian and creates noisy diffs; the rest of `.obsidian/` is fine to keep if you want your settings backed up.
- `graphify-out/` is generated — see below for whether you want it in git at all.

**The critical part: secrets.** Vaults accumulate API keys, tokens and SSH keys as stray notes. These must never reach GitHub, even in a private repo. Check what is already tracked:

```bash
cd ~/Documents/obsidian-vault
git ls-files | grep -iE 'key|token|secret|\.env'
```

If anything sensitive shows up, stop tracking it (keeping the local file) and add it to `.gitignore`:

```bash
git rm --cached <file>
echo "<file>" >> .gitignore
```

If a secret was already pushed, consider it compromised and rotate it — history keeps it forever otherwise.

### Step 2: Create a private repo and push
Keep the backup repo **private**. From the vault folder:

```bash
git init                          # skip if the vault is already a repo
git add -A
git commit -m "Vault backup"
gh repo create obsidian-vault --private --source=. --push
```

That last line creates the private repo on GitHub, adds it as `origin`, and pushes. Any existing history is preserved.

### Step 3: Back up from inside Obsidian
The [Obsidian Git](https://github.com/Vinzent03/obsidian-git) community plugin automates the whole loop without touching a terminal:

1. Settings → Community plugins → Browse → install **Obsidian Git**, enable it.
2. In its settings, set *Auto backup interval* (e.g. 10 minutes) and *Auto pull interval*.
3. Optionally enable *Push on backup* so commits reach GitHub automatically.

Now every writing session becomes a commit, and the plugin's *File history* view gives per-note versioning inside Obsidian.

### Step 4: Decide about the graph
The graph is derived data — the vault is the source of truth. Two workable strategies:

- **Regenerate on restore (recommended):** keep `graphify-out/` ignored. After cloning the vault on a new machine, reinstall Hermes (Part 1), run `/graphify`, and the graph comes back. Since extraction is cached only locally, a rebuild costs a few credits — the price of simplicity.
- **Snapshot the graph:** if you want the exact graph versioned, commit `graphify-out/graph.json` and `GRAPH_REPORT.md` but keep the HTML visualization ignored — it is large and rebuildable:

```gitignore
graphify-out/*
!graphify-out/graph.json
!graphify-out/GRAPH_REPORT.md
```

### Step 5: Restore
On a new machine:

```bash
gh repo clone <you>/obsidian-vault ~/Documents/obsidian-vault
```

Then open the folder as a vault in Obsidian, reinstall Hermes with the GLM key from [[Knowledge Base - Part 1 - Setup|Part 1]], and run `/graphify --update` (or a full `/graphify` if you kept the graph ignored). Between the git history and the graph cache of extracted concepts, nothing is lost but the machine.

### Tips
- Push after meaningful writing sessions even with automation — the plugin backs up on a timer, but a manual *Backup* (command palette) before closing the laptop never hurts.
- Check the repo size occasionally with `du -sh .git` — a vault with years of pasted images can grow. Keep large binaries out of notes, or move old attachments to cold storage.
- A private repo protects against accidents, not subpoenas — for sensitive notes, consider encrypting before committing or keeping them out of the vault entirely.
