# Phase 4 — CLAUDE.md Generation & Optimization

## Context

CLAUDE.md is the project's interface with Claude Code. It tells future Claude Code
sessions everything they need to know to work effectively in this codebase — project
purpose, architecture, conventions, commands, and gotchas. A well-written CLAUDE.md
dramatically improves the quality of Claude Code's output on subsequent tasks.

This phase generates CLAUDE.md from the refactored codebase, using all the artifacts
produced in previous phases.

## Prerequisites

- Completed Phases 0–3:
  - `INVENTORY.md`
  - `ARCHITECTURE_BEFORE.mermaid`
  - `PROTECTED_PATHS.md`
  - `PLAN.md` (especially Section 7 — CLAUDE.md recommendations)
  - `REFACTOR_LOG.md`
  - `CLEANUP_LOG.md`
- Refactored codebase on `claudify/refactor` branch
- Claude Code session with **Sonnet** model

## Instructions

Run the following in Claude Code:

```
Claudify Phase 4: Generate CLAUDE.md for this codebase.

Read PLAN.md Section 7 (CLAUDE.md recommendations), INVENTORY.md, PROTECTED_PATHS.md,
REFACTOR_LOG.md, and CLEANUP_LOG.md. Also scan the current codebase structure to
reflect the post-refactor state (not the pre-refactor inventory).

Generate CLAUDE.md at the project root with these sections:

## 1. Project overview

One paragraph explaining:
- What this project does (domain, purpose)
- Who it's for (end users, internal tool, library)
- The core problem it solves

Keep it concrete. "A Python toolkit for backtesting swing trading strategies using
historical market data" — not "A data analysis project."

## 2. Architecture

Brief description of the project structure after refactoring:
- Top-level directory layout with one-line descriptions per directory
- How modules relate to each other (data flow, not just imports)
- Key design decisions and why they were made

Do NOT paste the full file tree. Focus on the conceptual structure — what a developer
needs to understand to navigate the codebase. Reference ARCHITECTURE_BEFORE.mermaid
(or the post-refactor diagram if Phase 7 has run) for the visual version.

## 3. Key modules

For each core module (not every file — the important ones):
- What it does
- Key public functions/classes
- What it depends on and what depends on it

This is the "map" that lets Claude Code (or a human) know where to look for things.

## 4. Getting started

Practical commands to get the project running:
- Prerequisites (Python version, system dependencies)
- Installation (`pip install`, `npm install`, etc.)
- Configuration (env vars, config files)
- Running the project (entry points, CLI commands)
- Running tests (if they exist)

Use exact commands, not descriptions. `pip install -r requirements.txt` — not
"install the dependencies."

## 5. Coding standards

If /zenify-codebase is available, reference it:
"This project follows Type-Rich Minimalism standards. See /zenify-codebase."

If unavailable, embed the core rules:
- Type hints on all function signatures
- verb_noun function naming, is_/has_/should_ boolean prefixes
- Contextual docstrings explaining business logic (Google Style)
- Guard clauses over nested if/else
- Chesterton's Fence: don't remove defensive logic without proof it's redundant

## 6. Protected paths

List paths that should not be read or modified by Claude Code, with brief
explanations of why:
- "db/raw/ — raw research data, never modify"
- ".env — contains API keys"

Pull from PROTECTED_PATHS.md but add the human-readable "why" for each.

## 7. Domain terminology

If the project uses domain-specific terms (financial jargon, API-specific names,
Hebrew terms, etc.), define them here. This prevents Claude Code from misinterpreting
variable names or business logic.

## 8. Common tasks

A quick-reference for frequent development tasks:
- "Add a new data source: ..."
- "Run a backtest: ..."
- "Deploy to production: ..."

Only include tasks that are relevant to this specific project. If you're unsure
what tasks are common, ask the user.

---

## Writing style for CLAUDE.md

- Be concise. Claude Code reads this every session — bloat wastes context window.
- Be specific. Exact commands, exact paths, exact module names.
- Be current. Reflect the post-refactor state, not the original messy structure.
- No boilerplate. Skip sections that have nothing useful to say.
- Optimize for Claude Code, not for GitHub visitors. README.md (Phase 5) handles
  the human-facing documentation.

After generating CLAUDE.md, commit it:
`git add CLAUDE.md && git commit -m "claudify: add CLAUDE.md"`
```

## Success criteria

After this phase completes, verify:

- [ ] `CLAUDE.md` exists at the project root
- [ ] It reflects the post-refactor codebase structure (not the pre-refactor state)
- [ ] Protected paths are listed with explanations
- [ ] Getting started commands are exact and runnable
- [ ] Coding standards section is present (referencing zenify or embedding the rules)
- [ ] No bloat — each section earns its place
- [ ] Committed on the `claudify/refactor` branch
