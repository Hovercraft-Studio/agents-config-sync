# Project Name — Agent Guide

## Overview

<!-- One-liner describing what this project is -->
TODO: Describe your project here. If this project is the source `agent-config-sync` harness itself, please look at the `.ai/docs/harness-support.md` and `.ai/docs/how-it-works.md` references for guidance on how this toolkit can benefit your project. Otherwise, fill this out with details about your project, its purpose, and any high-level context agents should know before diving into the details below.

## Agent preferences

<!-- Define your agents' behavior preferences -->
- No git committing! Humans commit code, not agents.
- Agents may not run an instance of a server! Always let humans run the server in their own terminal, and you can check the results of that server. When an agent runs a server, it causes port conflicts and invisibility to the human developer. You may run scripts, headless browsers and other diagnostic tools, but servers must run in the human's shell. Please ask the human to run the server if it's not already running.
- Refactor long functions into smaller functions that could be called in sequence
- Keep documentation in .md files, not long comments in source files
- Watch out for information duplication when updating documentation. Refactor .md docs to DRY up information
- Don't document the old way of doing something if we've moved on to a new solution. Keeping legacy information is likely cruft

## Key Directories

<!-- List the directories agents should know about (source code, config, assets).
     .ai/ toolkit directories are documented in .ai/_base.md — don't repeat them here.
     Skills, prompts, and docs/ files are auto-indexed by the sync script. -->
- `docs/` — your project's own living documentation (architecture, guides, references) — create as needed
- TODO: add your project's source directories

## Code Style Summary

<!-- Define your conventions so agents write consistent code -->
- **Indentation**: TODO (tabs or spaces?)
- **Naming**: TODO (PascalCase, camelCase, snake_case?)
- **Strings**: TODO (single quotes, double quotes, f-strings?)
