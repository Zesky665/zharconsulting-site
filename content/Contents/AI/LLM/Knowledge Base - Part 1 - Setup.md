---
aliases:
  - Knowledge Base Setup
  - Obsidian and Hermes setup
tags:
  - AI
  - LLM
  - Local_LLM
  - Tools
---

This is Part 1 of building a local knowledge base on top of my Obsidian vault, using the Hermes agent to process it. This part covers the setup on macOS: installing Obsidian, installing Hermes, and wiring Hermes up to a Mistral coding plan token. [[Knowledge Base - Part 2 - Graphify|Part 2]] covers turning the vault into a knowledge graph with graphify.

### Why these two?
Obsidian is where the knowledge lives, as plain Markdown files. Hermes is the [open-source, self-hosted AI agent by Nous Research](https://github.com/NousResearch/hermes-agent) that does the processing. Since everything is local files, the two compose well: the agent reads and writes the same vault I edit by hand.

### Install Obsidian
1. Download the installer from [obsidian.md/download](https://obsidian.md/download) and drag it to Applications. Or with Homebrew:
```bash
brew install --cask obsidian
```
2. Open Obsidian and create a vault — this is just a folder of `.md` files on disk. Mine lives at `~/Documents/obsidian-vault`.
3. If you are new to Obsidian, the [official help docs](https://help.obsidian.md/) are excellent, and the vault itself is [fully documented](https://help.obsidian.md/Extending+Obsidian/Obsidian+API) if you ever want to script around it.

That is the whole install. No account needed.

### Install Hermes
Docs: https://hermes-agent.nousresearch.com/docs/

The installer handles everything (Python 3.11 via `uv`, Node.js v22, ripgrep, ffmpeg, the repo clone, a venv, and the global `hermes` command). The only hard prerequisite on macOS is `git` — check with `git --version`.

Option A — Desktop app (recommended):
Download the [Hermes Desktop installer](https://hermes-agent.nousresearch.com/) and run it. This installs both the desktop app and the CLI.

Option B — Command line only:
```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```
Add the desktop app later any time with `hermes desktop`.

Verify the install:
```bash
source ~/.zshrc
hermes doctor    # diagnostics — reports anything missing and how to fix it
hermes           # start chatting
```
If you get `hermes: command not found`, reload your shell or check that `~/.local/bin` is on your `PATH`. Everything Hermes-related lives under `~/.hermes/`.

### Set up Hermes with a Mistral coding plan token
Instead of pay-as-you-go API keys, Mistral offers a [coding plan subscription](https://console.mistral.ai/) — a flat monthly token that works with any coding agent through their OpenAI-compatible endpoint. This is the cheapest way to run Hermes on a frontier model.

1. Subscribe to the plan: log in to [console.mistral.ai](https://console.mistral.ai/), go to **Billing**, and pick a coding plan that fits your usage.
2. Copy your token from the console. Treat it like a password — it is billed per use.
3. Point Hermes at Mistral. Hermes speaks any custom OpenAI-compatible endpoint via the model picker:
```bash
hermes model
```
Choose the custom OpenAI-compatible endpoint option and enter:
- **Base URL:** `https://api.mistral.ai/v1`
- **API key:** your coding plan token
- **Model:** a Mistral coding model, e.g. Mistral Medium 3.5 (see the [model docs](https://docs.mistral.ai/) for the current list)

Keys are stored in `~/.hermes/.env`. You can also set them directly:
```bash
hermes config set MISTRAL_API_KEY <your-token>
```
4. Check what is active:
```bash
hermes config get model
```

### Sanity check
Run a quick chat in your vault folder to confirm the model answers:
```bash
cd ~/Documents/obsidian-vault
hermes
```
Ask it something trivial, and once it responds you are done with the setup. Continue with [[Knowledge Base - Part 2 - Graphify|Part 2 — Graphify]] to turn the vault into a queryable knowledge graph.
