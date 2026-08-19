# Agentic Coding for Humans

This document is a practical guide for humans working in our preferred agentic coding workflow. It is partly an onboarding guide and partly a team standards document. 

An "agent harness" refers to a tool like Claude and Copilot, which have LLM models that can generate code, but also have a larger set of tools that allow them to interact with the filesystem, the internet, and other tooling. This extends our vanilla agentic coding setups with knowledge and capabilities needed to operate effectively within our projects.

Core ideas are:

- **Skills**, **prompts**, **MCP servers**, and **/docs** are the main tools that we use to guide our agents. They are:
  - A way to ensure that *my* agent and *your* agent behave the same way on the same repo, even if we are using different tools (Claude vs Copilot harness, or VSCode vs Zed editor)
  - The shared memory of a project
  - Guidance for agents' behavior (ex: don't commit or push autonomously, use Convex for database access, etc.)
  - Guardrails that align agents to our desired coding patterns & preferences
  - Hallucination mitigation
  - Extra superpowers via MCP servers
- When we start a new agent chat session, the agent should be able to recover the most important (shared) context from the repo itself, **not from a previous chat's memory that's hidden on our individual machines**. 
- We support multiple agentic stacks, since developers have different preferences and constraints. We optimize around a repeatable working style, with this project's `.ai/` patttern as the portable source of truth for instructions, skills, prompts, and MCP configuration. That lets the team move between GitHub Copilot, Claude, Antigravity CLI, Codex, `.pi`, and even locally-hosted setups while providing consistent behavior and shared context for (almost) any agentic coding workflow.

## Setting Up Agent Context in Your Project

The minimal setup for any project is simple: create an `AGENTS.md` file at the root with project-specific instructions for agents (coding preferences, key directories, patterns). Point `CLAUDE.md` to it with `@AGENTS.md`. This gives agents shared context.

For more robust setups with skills, prompts, and MCP servers that work consistently across different tools (Claude, Copilot, Codex, etc.), this repository provides a sync toolkit. See **[.ai/README.md](../README.md#quickstart)** for setup details and **[.ai/docs/how-it-works.md](how-it-works.md)** for architecture.

## Documentation as Operational Memory

In agentic development, documentation serves as **operational memory** for future sessions. Without it, agents lose context between sessions and must rebuild understanding from scratch—costing time and tokens.

### The Memory Layer

Store knowledge in markdown files within the repository:
- **Project instructions** — coding preferences, key directories, patterns (`AGENTS.md` or `.ai/AGENTS.md`)
- **Skills** — reusable domain knowledge and workflows (`.ai/skills/`)
- **Prompts** — repeatable slash commands (`.ai/prompts/`)
- **Living documentation** — architecture, guides, learnings (`docs/`)

This ensures both humans and agents across the team access the same knowledge consistently.

### Capturing Learnings

When an agent completes a difficult task or solves a tricky problem, capture the learning:
- **Repeatable pattern?** → Create a skill (`.ai/skills/`)
- **One-off lesson?** → Add to `/docs/learnings/`
- **Project preference?** → Update `AGENTS.md`

Ask the agent to update relevant docs/skills/prompts in the same commit as the code change. This ensures future sessions can solve similar problems without repeating the same debugging cycles.

### When to Create `docs/`

For smaller codebases, `AGENTS.md` may be sufficient. Once a project grows in complexity, create a structured `docs/` folder maintained as part of engineering work. Run `/harness-docs-setup` to scaffold an opinionated structure (architecture, commands, design patterns, etc.).

### Maintenance Workflow

Keep documentation current as the project evolves:

1. **Update as you go** — When behavior changes or new patterns emerge, update the corresponding doc in the same commit
2. **Run `/update-docs`** — After significant changes, use this prompt to audit and update affected documentation
3. **Capture learnings** — After completing difficult tasks, document the solution
4. **Prevent bloat** — Refactor docs to appropriate locations; avoid duplication
5. **Validate discoverability** — In fresh sessions, verify agents can find what they need

This turns documentation from passive reference into active runtime context for every new agent session.

## Team Defaults

- GitHub Copilot and Claude are the primary tools we expect teammates to use day to day.
- Antigravity CLI, Codex, `.pi`, and even local-model setups are supported because the harness is meant to stay vendor-agnostic.
- Open source and local models, including `.pi` wired to local backends such as Gemma, are useful for experimentation, privacy-sensitive workflows, offline work, and cost control, and they can use the same instructions and operating standards as the commercial tools. See [.ai/docs/local-llm-hosting.md](.ai/docs/local-llm-hosting.md) for local hosting notes in this repo.

## Human Expectations and Ownership of Code

- Since we're not necessarily writing code line-by-line like we used to, we need to spend *much* more time reviewing and testing our code. We don't have the mental model we'd have gained by planning and writing the code from scratch, so in order to not beclown ourselves 🤡 in front of our peers and clients by launching buggy software, we need to do a *lot* more testing with AI-generated code. Velocity is useless without testing for quality. 
- Agents still write bad code and don't follow our patterns and preferences. The bigger the project and the longer the lifecycle, the bigger the potential problem. The less we understand about our code, the more ownership humans need to take in reviewing and refactoring the AI-generated code.
- Breaking our work into smaller chunks makes it easier to review and test. This helps us build a better mental map of what we've built.
- AI tools can dramatically accelerate development, but it is not a substitute for human judgment and expertise in the engineering process. Humans must remain responsible for the quality and maintainability of the code.
- With the acceleration of code generation, we can gain an entirely new layer of testing and error-catching by quickly building small demo apps, simulations of larger systems, automated smoke tests, and watchdog scripts for production environments. These efforts can offset some of the risks introduced with faster AI-powered development cycles.
- Ask yourself: How is the UX? How is the design? Can we make improvements with the extra development velocity and capabilities provided by AI? Our work should be *even better* than it could have been pre-AI. Yes it can get done faster, but can it become more robust, trustworthy, and impressive to the user? This is what will add value and separate our work from vibe-coded slop.

## Practical Tips

### Session Restarts

After updating agent configuration files, restart your agent session:
- **Copilot**: Reload the VS Code window or start a fresh chat
- **Claude**: Start a new session (context is indexed at session start)

If a skill or prompt appears missing, restart before troubleshooting.

### MCP Servers

MCP servers provide extended capabilities beyond the base agent (database access, documentation search, external APIs). Behavior varies by tool:
- **Claude**: Shows approval prompts when a server is first accessed
- **Copilot**: Check the tools UI or run `MCP: List Servers` from the command palette

See **[.ai/docs/harness-support.md](harness-support.md)** for per-tool MCP configuration details.

## Future Enhancements

Potential additions to our stack:
- https://github.com/TencentCloud/TencentDB-Agent-Memory/
