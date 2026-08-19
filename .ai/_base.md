## Core Principle: Documentation is Infrastructure (if docs exist)

**If this project maintains documentation** (either `.ai/docs/`, project-level `docs/`, or both), then **harness documentation and agent context are part of your system infrastructure**. Whenever you change code, patterns, or agent behavior, update the corresponding docs in the same change—agents read these docs to understand how to behave consistently.

Use the `/update-docs` prompt to audit and fix docs after any change. The sync harness ensures all agents see the same current instructions.

**If this project has no docs**, skip this discipline. The sync harness works fine without it.

---

## How `.ai/` Works

The `.ai/` directory is the **Agents Config Sync** toolkit — the single source of truth for AI agent configuration in this repository, synced to every harness.

* **Never edit generated files directly** (`AGENTS.md`, `CLAUDE.md`, `.github/copilot-instructions.md`, `.agents/context/AGENTS.md`, `.mcp.json`, etc.).
* **To update context**, edit `.ai/AGENTS.md` or files under `.ai/skills/`/`.ai/prompts/` and run `node .ai/scripts/sync.js` to regenerate.
* For sync architecture details and file-mapping tables, see [.ai/docs/how-it-works.md](.ai/docs/how-it-works.md) and [.ai/README.md](.ai/README.md).

### Authoring Convention: Root-Relative Links
All cross-references inside `.ai/` source files (links to skills, docs, or other markdown) **must use root-relative paths**, e.g.:
`See [docs/systemArchitecture.md](docs/systemArchitecture.md) and [.ai/skills/code-reviewer.md](.ai/skills/code-reviewer.md).`

---

## Documentation Maintenance

* **Do not edit `.ai/docs/`**: This directory documents the sync toolkit itself. Only toolkit maintainers should update it.

* **Update your project's custom docs instead**:
  1. **`.ai/skills/`** — Domain knowledge for *this* project (APIs, patterns, workflows).
  2. **`.ai/AGENTS.md`** — Project-specific context (key dirs, style, team).
  3. **`docs/`** (at root, if present) — Living project docs (architecture, specs, guides).
  4. **`README.md`** (root) — Entry point for humans.

* **Key principle**: Code changes that alter behavior must update the corresponding docs under `docs/` or `.ai/` in the same work.

* **Agent learning pattern (optional)**: Some projects maintain a troubleshooting log where agents record non-obvious solutions after debugging. If your project adopts this pattern, agents should:
  - Consult existing entries before deep debugging
  - Record any fix that required significant investigation or multiple iterations
  - Keep entries concise: **Symptom → Root cause → Fix**
