# `.ai/` — Cross-Harness Agent Configuration

This .ai/ folder is the **source of truth** for AI agent context across Claude Code, OpenAI Codex, VS Code Copilot, Antigravity CLI, Cursor, and others. A zero-dependency Node.js sync script in [.ai/scripts/sync.js](.ai/scripts/sync.js) fans these source files out to every harness's expected paths so you author once and every tool sees the same instructions.

`sync.js` is module-mode safe: it runs in projects with or without `"type": "module"` in `package.json`.

If you've just dropped `.ai/` into a new project, start with the [Quickstart](#quickstart) below. If you're trying to add a skill, prompt, or MCP server, jump to [Adding things](#adding-things).

You can find the latest version of this project at https://github.com/Hovercraft-Studio/agents-config-sync

---

## What you edit vs. what's generated

The harness only works if you stay on the right side of this line.

### Edit these (sources)

| Path | Purpose |
|---|---|
| `.ai/AGENTS.md` | Project-specific agent instructions (what the project is, key dirs, project doc links) |
| `.ai/mcp-servers.json` | MCP server definitions (ships pre-wired to `@modelcontextprotocol/server-everything` for smoke-testing — delete or replace once you add real servers) |
| `.ai/skills/<name>.md` | Domain knowledge agents load when relevant, or manually invoked by the user |
| `.ai/prompts/<name>.md` | Slash commands you invoke explicitly |

### Don't edit these (harness internals)

These ship with the harness and are maintained by the agents-config-sync project. Touch them only if you're upstreaming improvements to the harness itself — adopters should leave them alone so they can pull future updates cleanly.

| Path | Why |
|---|---|
| `.ai/scripts/sync.js` | Sync engine — only modify if extending sync behavior |
| `.ai/.sync-manifest.json` | Generated state; regenerated on every sync |
| `.ai/docs/*.md` | Harness reference docs (per-harness support, test instructions, setup notes) |
| `.ai/_base.md` | Instructions regarding how the harness sync works |
| `.ai/README.md` | This file — documents the harness itself |

### Never edit these (generated harness targets)

These are produced from `.ai/` sources on every sync. They are **gitignored** and any direct edits will be overwritten.

| Path | Harness |
|---|---|
| `CLAUDE.md`, `AGENTS.md` | Claude Code; Codex + Antigravity CLI (both read `AGENTS.md`) |
| `.github/copilot-instructions.md` | VS Code Copilot |
| `.agents/context/AGENTS.md` | Codex / Antigravity `.agents/` layout |
| `.claude/skills/<name>/SKILL.md` | Claude Code skills + prompts |
| `.claude/commands/<name>.md` | Claude Code commands (deprecated location, still written for CLI compat) |
| `.agents/skills/<name>/SKILL.md` | Codex + Antigravity skills + prompts (real file copies — Codex selectors don't follow symlinks) |
| `.agents/mcp_config.json` | Antigravity CLI MCP servers (generated JSON, `serverUrl` schema) |
| `.github/skills/<name>/SKILL.md` | Copilot skills |
| `.github/prompts/<name>.prompt.md` | Copilot prompts |
| `.mcp.json` | Claude Code + Copilot MCP config (symlink to `.ai/mcp-servers.json`) |
| `.pi/mcp.json` | `.pi` agent harness MCP config (symlink to `.ai/mcp-servers.json`) |
| `.codex/config.toml` | Codex MCP config (generated TOML) |

Whenever you change a source, run `node .ai/scripts/sync.js` (or let one of the [automatic triggers](#when-the-sync-runs-automatic-triggers) handle it).

---

## Quickstart

> If you have an existing CLAUDE.md or AGENTS.md, rename them temporarily to get the sync set up, remove the original files from git with a commit, then copy their contents into `.ai/AGENTS.md` and let the sync regenerate them.

### Automated init

Run the following command from this repo to automate the manual steps below. This script copies and/or merges the core toolkit, and runs the sync in the target. It also pre-checks for human-authored files at paths the sync engine manages (`CLAUDE.md`, `AGENTS.md`, `.mcp.json`, etc.) and exits with a warning instead of silently proceeding if it finds any. Add `--dry-run` to preview, `--force` to overwrite an existing `.ai/` and bypass that check, or `--no-sync` to skip the final sync run.

```bash
npm run ai-init -- <path-to-target-project> [--no-sync] [--dry-run] [--force]

# simple example for a sibling project
npm run ai-init -- ../sibling-project
```

> **If `.ai/` already exists in the target**, the script does **not** copy or overwrite anything and does **not** run the sync — it prints a loud warning and tells you to rerun with `--update` (see below). This prevents an accidental no-op sync against a stale toolkit.

To update a project that already has an older `.ai/` toolkit, use `--update`: it refreshes only the harness-internal files (`.ai/scripts/`, `.ai/_base.md`, `.ai/docs/`) and adds any new example skills/prompts that don't exist yet, without touching `.ai/AGENTS.md`, `.ai/mcp-servers.json`, or any skill/prompt file you already have. You can add `--force` alongside `--update` to also overwrite example skills/prompts you haven't customized.

```bash
npm run ai-init -- <path-to-target-project> --update
```

### Manual steps:
#### Step 1: Copy the core toolkit

Copy these files/folders from agents-config-sync to your target project:


| Source | Target | Notes |
|--------|--------|-------|
| `.ai/` | `<your-project>/.ai/` | **Copy the entire directory.** This is the source of truth. |
| `.githooks/` | `<your-project>/.githooks/` | Optional. Git hooks auto-sync on pull/checkout. |
| `.gitattributes` | `<your-project>/.gitattributes` | Optional. If your project already has this, merge the entries. |
| `.vscode/` | `<your-project>/.vscode/` | Optional. VS Code settings and tasks for the project. |

#### Step 2: Merge these files into your project

| Source | Target | What to do |
|--------|--------|-----------|
| `.gitignore` | `<your-project>/.gitignore` | Find the `# Cross-AI Tooling` comment in agents-config-sync and copy that entire block into your `.gitignore`. |
| `package.json` | `<your-project>/package.json` | Copy the `scripts` block: - `"ai-sync": "node .ai/scripts/sync.js"` and `"ai-watch": "node .ai/scripts/sync.js --watch"`. Add `"postinstall": "node .ai/scripts/sync.js"` if not present. |
| `.vscode/tasks.json` | `<your-project>/.vscode/tasks.json` | Create or update with the `ai-sync` task below (create `.vscode/` folder if needed). |


#### Step 4: Run the sync

```bash
node .ai/scripts/sync.js --watch
# or
npm run ai-watch
```

Or if you already have Node installed and are running npm anyway:
```bash
npm install  # postinstall hook runs sync automatically
```

> **Gitignore note**: The generated files (`CLAUDE.md`, `AGENTS.md`, `.github/copilot-instructions.md`, etc.) are gitignored. Teammates need to run `node .ai/scripts/sync.js` (or `npm install`) after cloning to regenerate them.

#### Step 5: (Optional) Enable git hooks

If you copied `.githooks/`:
```bash
git config core.hooksPath .githooks
```

This auto-syncs whenever you `git pull` or `git checkout`.

#### Step 6: Verify that it works

> Ask any agent: **"Is my agent harness set up correctly?"**

The [validate-harness-sync](.ai/skills/validate-harness-sync.md) skill will load and produce a PASS/FAIL report. Or follow the manual checklist in [.ai/docs/test-instructions.md](.ai/docs/test-instructions.md).

#### Step 7: (Recommended) Set up docs tree

Run `/harness-docs-setup` in any agent (Claude Code, Copilot, Codex, Antigravity CLI). This creates an opinionated docs structure (`ARCHITECTURE.md`, `COMMANDS.md`, `DESIGN.md`, etc.) that gives agents deep, navigable context.

See [.ai/prompts/harness-docs-setup.md](.ai/prompts/harness-docs-setup.md) for details, or build it by hand using that prompt's spec.

---

## Adding project-specific tools

### Adding skills

Skills are domain knowledge files that agents load when their `description` matches the user's task. Create a flat markdown file in `.ai/skills/`:

```markdown
<!-- .ai/skills/my-domain.md -->
---
name: My Domain Knowledge
description: Use when working on [domain]. Do NOT use for [anti-trigger].
---

## When to Use

Load this skill when working on [describe the domain].

## Key Patterns

- Pattern 1: description
- Pattern 2: description
```

The sync script reads `.ai/skills/`, `.ai/prompts/`, and the project's `docs/` folder (recursive) and builds an index section (link + description per file) that is injected between `.ai/AGENTS.md` and `.ai/_base.md` in every composed output (`CLAUDE.md`, `AGENTS.md`, etc.). The index also lists MCP servers from `.ai/mcp-servers.json` so agents know which tool servers to expect (add an optional `description` key per server for a friendlier entry; otherwise the launch command or URL is shown). Descriptions come from frontmatter; docs without frontmatter fall back to their first `# ` heading. 

**Description quality matters more than skill content.** Front-load the trigger condition and add anti-triggers ("Do NOT use for …"). See [.ai/docs/harness-support.md](.ai/docs/harness-support.md) for details.

### Adding prompts (slash commands)

Prompts become reusable slash commands you can invoke in chat. This repo already ships some, with a basic example to confirm that prompts are working — [.ai/prompts/example-command.md](.ai/prompts/example-command.md) — whose only job is to print "Hello World" with emojis. Running `/example-command` (or the harness-specific equivalent below) is the fastest way to confirm prompts are wired up end-to-end before you author your own.

Create new prompts as flat markdown files in `.ai/prompts/`:

```markdown
<!-- .ai/prompts/my-prompt.md -->
---
name: my-prompt
description: One-line description of what this prompt does.
---

Body of the prompt sent to the model when invoked.
```

After sync, `example-command` should become available as a slash command in the chat interface of the harness. You maybe need to restart the harness or refresh the chat interface for the new command to appear.

### Adding MCP servers

The repo ships with `.ai/mcp-servers.json` pre-wired to Anthropic's reference test server, [`@modelcontextprotocol/server-everything`](https://www.npmjs.com/package/@modelcontextprotocol/server-everything):

```json
{
  "mcpServers": {
    "everything": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-everything"],
      "autoStart": true
    }
  }
}
```

This is a no-install canary — `npx -y` fetches it on first run. Use it to verify MCP wiring works in every harness (ask any agent to call the `echo` or `add` tool from the `everything` server and watch for the tool-confirmation prompt). Once your own servers are wired up, delete the `everything` entry or replace it.

Add your servers under the `mcpServers` key:

```json
{
  "mcpServers": {
    "my-docs-server": {
      "command": "node",
      "args": ["path/to/server.js"],
      "env": {}
    }
  }
}
```

> **Empty source = no generation.** When `mcpServers` is empty (or the source file is missing), the sync **skips all four destinations** and cleans up any previously-generated stubs it still owns. User-owned files at those paths are preserved. This keeps fresh repos free of empty MCP config files until you actually have servers to wire up.

See [.ai/docs/harness-support.md](.ai/docs/harness-support.md) for per-harness config details and [.ai/docs/test-instructions.md](.ai/docs/test-instructions.md) for how to verify each harness sees the servers.

---

## How the Sync Script Works

The sync engine ([.ai/scripts/sync.js](.ai/scripts/sync.js)) is a zero-dependency Node.js script that composes agent instructions, links skills/prompts, and syncs MCP configuration across all supported harnesses.

**High-level operation:**
1. Creates necessary directories for each tool
2. Builds an auto-generated index of skills, prompts, docs, and MCP servers
3. Composes agent instructions from `.ai/AGENTS.md` + index + `.ai/_base.md`
4. Links or copies skills and prompts to each harness's expected locations
5. Syncs MCP configuration (symlinks or generates tool-specific formats)
6. Cleans up stale generated files

For detailed architecture, sync engine behavior, git configuration, and adoption strategies, see **[.ai/docs/how-it-works.md](.ai/docs/how-it-works.md)**.

---

## When the Sync Runs (Automatic Triggers)

The sync runs automatically at key lifecycle moments:

| Trigger | How to Enable |
|---------|---------------|
| **VS Code workspace open** | `.vscode/tasks.json` — First time requires clicking "Allow Automatic Tasks" notification |
| **Git pull / merge / checkout** | Run once: `git config core.hooksPath .githooks` |
| **npm install** | Automatic via `"postinstall"` in `package.json` |
| **Manual** | `npm run ai-sync` or `node .ai/scripts/sync.js` |
| **Watch mode** | `npm run ai-watch` — Live reload during authoring |

---

## Cursor Compatibility

Cursor reads from `.cursor/rules/` for project rules. The sync script doesn't generate Cursor files by default, but you have options:

1. **Manual symlink** (recommended if using Cursor):
   ```bash
   # Link your skills into Cursor's rules directory
   mklink /D .cursor\rules .ai\skills   # Windows
   ln -s .ai/skills .cursor/rules       # macOS/Linux
   ```

2. **Cursor reads `AGENTS.md`** at the repo root, which the sync already generates from your `.ai/AGENTS.md` + `.ai/_base.md`.

3. **Add Cursor targets to sync.js** — extend the `skillTargets` or add a new section if you want full automation.

---

## Authoring Convention: Root-Relative Links

All cross-references in `.ai/` source files (including this README) must use **root-relative paths**:

```markdown
See [.ai/docs/harness-support.md](.ai/docs/harness-support.md) and
[.ai/skills/my-skill.md](.ai/skills/my-skill.md).
```

Symlinks resolve relative paths from the *link's* location, not the source file's. Root-relative links work from both `AGENTS.md` (at the repo root) and `.github/copilot-instructions.md` (one level deep). They may appear broken when browsing inside `.ai/` in an editor — that's expected.

---

## Project Structure

```
├── .ai/                          ← Source of truth (this folder)
│   ├── README.md                 ← You are here
│   ├── AGENTS.md                 ← Project-specific agent instructions (skills/prompts index injected at sync time)
│   ├── _base.md                  ← Portable toolkit instructions (do not edit as adopter)
│   ├── mcp-servers.json          ← MCP server definitions (optional)
│   ├── scripts/sync.js           ← Sync engine (zero dependencies)
│   ├── docs/                     ← Deeper reference docs
│   │   ├── harness-support.md    ← Per-harness reference
│   │   ├── test-instructions.md  ← Verification steps
│   │   └── how-it-works.md       ← Architecture & design rationale
│   ├── skills/                   ← Domain knowledge (flat .md files)
│   │   └── validate-harness-sync.md
│   └── prompts/                  ← Slash commands (flat .md files)
│       ├── example-command.md
│       ├── harness-docs-setup.md
│       └── update-docs.md
├── .githooks/                    ← Git hooks (auto-sync on pull/checkout)
│   ├── post-merge
│   └── post-checkout
├── .vscode/tasks.json            ← VS Code auto-sync on workspace open
├── README.md                     ← Thin stub pointing into .ai/
├── package.json                  ← npm aliases (ai-sync, ai-watch, postinstall)
├── .gitignore                    ← Ignores all generated targets
└── .gitattributes                ← LF line endings for git hooks
```

### Generated (gitignored) outputs

```
├── AGENTS.md                     ← Read by Codex, Antigravity CLI, Cursor, Amp, generic agents
├── CLAUDE.md                     ← Read by Claude Code
├── .mcp.json                     ← Read by Claude Code + VS Code Copilot
├── .codex/
│   └── config.toml               ← Codex MCP servers (generated TOML)
├── .agents/
│   ├── context/AGENTS.md         ← Codex / Antigravity .agents/ context layout
│   ├── mcp_config.json           ← Antigravity CLI MCP servers (serverUrl schema)
│   └── skills/<name>/SKILL.md    ← Codex + Antigravity skills + prompts (real files, run sync after clone)
├── .github/
│   ├── copilot-instructions.md   ← Read by VS Code Copilot
│   ├── prompts/<name>.prompt.md  ← Copilot slash commands
│   └── skills/<name>/SKILL.md    ← Copilot skills
└── .claude/
    ├── commands/<name>.md        ← Claude Code slash commands (deprecated location)
    └── skills/<name>/SKILL.md    ← Claude Code skills + prompts
```

---

## Design Principles

- **Vendor-agnostic**: One source, many targets. No lock-in.
- **Zero dependencies**: Plain Node.js — no npm install required for the sync itself.
- **Progressive disclosure**: Short root map → detailed docs via links.
- **Safe**: Never overwrites human files. Symlink or hash-verified copies only.
- **Cross-platform**: Identical behavior on macOS and Windows.

## Credits

- Authored by @cacheflowe
- `/docs` strategy adapted from the [Harness Engineering](https://openai.com/index/harness-engineering/) approach to AI-assisted development.
