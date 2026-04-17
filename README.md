# Ghost-Bridge

Cross-language IPC for Claude Code. Call Python, Ruby, Go, or Rust from TypeScript (and any other combination) — no HTTP, no ports, no zombies. JSON-RPC over stdin/stdout, fully typed, Ctrl+C safe.

---

## Installation

### Option A — Slash command in your own project (recommended, works today)

Copy the `.claude/commands/` folder from this repo into your project:

```bash
# inside your project root
cp -r /path/to/claude-bridge/.claude ./
```

Or clone Ghost-Bridge and symlink:

```bash
git clone https://github.com/gurudatta-patil/claude-bridge ~/.ghost-bridge-repo

# inside your project
mkdir -p .claude/commands
ln -s ~/.ghost-bridge-repo/.claude/commands/ghost-bridge.md .claude/commands/ghost-bridge.md
ln -s ~/.ghost-bridge-repo/.clone/commands/ghost-bridge-status.md .claude/commands/ghost-bridge-status.md
```

Then open Claude Code in your project and type:

```
/ghost-bridge typescript python image_processor 'resize images using Pillow' 'Pillow'
```

Claude will generate `.ghost-bridge/bridges/image_processor.py` and `image_processor.ts`, create the venv, and install dependencies — all in one shot.

---

### Option B — Install globally (available in every project)

```bash
git clone https://github.com/gurudatta-patil/claude-bridge ~/.ghost-bridge-repo

mkdir -p ~/.claude/commands
cp ~/.ghost-bridge-repo/.claude/commands/ghost-bridge.md ~/.claude/commands/
cp ~/.ghost-bridge-repo/.clone/commands/ghost-bridge-status.md ~/.claude/commands/
```

The commands are now available in every Claude Code session on your machine.

---

### Option C — MCP server (Phase 2, coming soon)

Once the MCP server is built, install it with one command:

```bash
claude mcp add ghost-bridge -- node ~/.ghost-bridge-repo/mcp-server/dist/index.js
```

This gives Claude Code a proper tool call (`generate_ghost_bridge`) that runs fully autonomously without a slash command prompt.

---

## Usage

### `/ghost-bridge` — generate a bridge

```
/ghost-bridge <source> <target> <name> '<capability>' '<dependencies>'
```

| Argument | Example |
|----------|---------|
| `source` | `typescript` `go` `python` `rust` |
| `target` | `python` `ruby` `go` `rust` `nodejs` |
| `name` | `image_processor` (snake_case) |
| `capability` | `'resize and watermark images'` |
| `dependencies` | `'Pillow, numpy'` (pip/gem/go module names) |

**Examples:**

```bash
# TypeScript app that needs Pillow
/ghost-bridge typescript python image_processor 'resize images' 'Pillow'

# Go service that needs a Ruby PDF library
/ghost-bridge go ruby pdf_generator 'generate PDFs from HTML' 'prawn'

# Python script calling a fast Rust number-cruncher
/ghost-bridge python rust stats_engine 'compute statistics on float arrays' ''
```

Claude will:
1. Read the matching template from `bridges/<source>-<target>/`
2. Fill in your capability and dependencies
3. Create `.ghost-bridge/bridges/<name>.<ext>` (sidecar + client)
4. Set up `.ghost-bridge/.venv` and install dependencies (Python targets)
5. Show you the build command (Rust/Go targets)
6. Print a usage snippet

---

### `/ghost-bridge-status` — inspect what's installed

```
/ghost-bridge-status
```

Shows all generated bridges, the Python venv version, and available bridge pairs.

---

## How it works

```
Your App (TypeScript)          Python sidecar
      │                              │
      │  spawn(.venv/bin/python)     │
      │─────────────────────────────▶│ {"ready":true}
      │◀─────────────────────────────│
      │                              │
      │  {"id":"uuid","method":"x"}  │
      │─────────stdin───────────────▶│
      │                              │  runs your logic
      │  {"id":"uuid","result":{}}   │
      │◀────────stdout───────────────│
      │                              │
      │  process.exit() / Ctrl+C     │
      │──── stdin EOF ──────────────▶│ exits immediately
```

No HTTP. No ports. No leftover processes. The child always dies with the parent.

---

## Supported language pairs

| Source \ Target | Python | Ruby | Go | Rust | Node.js |
|----------------|:------:|:----:|:--:|:----:|:-------:|
| **TypeScript** | ✅ | ✅ | ✅ | ✅ | — |
| **Go** | ✅ | ✅ | — | — | ✅ |
| **Python** | — | ✅ | ✅ | ✅ | — |
| **Rust** | ✅ | ✅ | ✅ | — | — |

---

## Project structure

```
bridges/          13 language pairs — templates, tests, edge cases, future scope
shared/           shared infrastructure per language (base classes, helpers)
languages/        per-language rules, signal contracts, OS notes
.claude/commands/ slash commands for Claude Code
info/wiki/        local-only architecture docs (git-ignored)
```

---

## Prerequisites by target language

| Target | Requirement |
|--------|------------|
| Python | Python 3.9+; `uv` optional (faster installs) |
| Ruby | Ruby 3.1+; `bundler` for gem management |
| Go | Go 1.21+ in PATH |
| Rust | Rust stable via `rustup`; first build takes ~30 s |
| Node.js | Node 18+ in PATH |
