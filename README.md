# Agents Config Sync

A vendor-agnostic configuration system for AI coding agents. Write your instructions, skills, and prompts **once** in `.ai/`, and a sync script fans them out to every harness (Claude Code, VS Code Copilot, Cursor, Antigravity CLI, Codex, and others).

## Quick Start

```bash
npm run ai-init -- ../your-project
```

This copies the `.ai/` toolkit into your project and runs the initial sync. Watch for warnings, and move any existing overlapping files to avoid conflicts.

Once this is in place, run the following to keep the harnesses in sync as you edit `.ai/` files. You can also run `npm run ai-sync` manually to sync on demand.

```bash
npm run ai-sync --watch
```

## Documentation

- **[.ai/README.md](.ai/README.md)** — Setup guide, adding skills/prompts/MCP, technical details
- **[.ai/docs/agentic-coding-for-humans.md](.ai/docs/agentic-coding-for-humans.md)** — Practical guide for teams using AI coding tools
- **[.ai/docs/how-it-works.md](.ai/docs/how-it-works.md)** — Architecture and design rationale
- **[.ai/docs/harness-support.md](.ai/docs/harness-support.md)** — Per-harness reference
- **[.ai/docs/test-instructions.md](.ai/docs/test-instructions.md)** — Verification steps

## What This Solves

Different AI coding tools expect configuration in different locations:
- Claude Code reads `CLAUDE.md`, `.claude/skills/`, and `.mcp.json`
- VS Code Copilot reads `.github/copilot-instructions.md`, `.github/skills/`, and `.mcp.json`
- Codex/Antigravity CLI read `AGENTS.md` and `.agents/`
- Cursor reads `AGENTS.md` or `.cursor/rules/`

Instead of maintaining parallel copies, you author **once in `.ai/`** and the sync generates the tool-specific files automatically. Symlinks are used where possible; files are copied when necessary.

## How It Works

1. Edit source files in 
  - `.ai/AGENTS.md`
  - `.ai/skills/`
  - `.ai/prompts/`
  - `.ai/mcp-servers.json`
2. Run `npm run ai-sync --watch` (or let automatic triggers handle it)
3. Every harness should see the same context in its expected location

See [.ai/README.md](.ai/README.md) for detailed setup and [.ai/docs/how-it-works.md](.ai/docs/how-it-works.md) for architecture.

