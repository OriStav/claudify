---
name: claudify
description: >
  Refactor and organize messy codebases (especially data research projects) into clean,
  well-documented repos optimized for Claude Code and human maintainability. Use this skill
  whenever the user mentions "claudify", "clean up my codebase", "organize my repo",
  "refactor for Claude Code", "make this codebase maintainable", or any request to take
  a messy/research-style project and turn it into a structured, documented codebase.
  Also trigger when a user asks to generate a CLAUDE.md, create a codebase overview,
  produce an architecture diagram from existing code, or prepare a repo for AI-assisted
  development. Even if the user just says "this code is a mess, help me organize it" —
  use this skill.
---

# Claudify

Transform messy, research-style codebases into clean, well-documented repositories
optimized for Claude Code workflows and human maintainability.

## Overview

Claudify works in 8 phases, each with a dedicated prompt template in `prompts/`.
The user runs them sequentially in Claude Code.

| Phase | Name | Model | Modifies files? |
|-------|------|-------|-----------------|
| 0 | Pre-flight | Opus | No |
| 1 | Architect | Opus | No |
| 2 | Refactor | Sonnet | Yes |
| 3 | Cleanup | Sonnet | Yes (deletions) |
| 4 | CLAUDE.md | Sonnet | Yes |
| 5 | README.md | Sonnet | Yes |
| 6 | Tests | Sonnet | Yes |
| 7 | Validation | Sonnet | No (report only) |

---

## Phase 0 — Pre-flight

**Goal**: Build a complete picture of the codebase before touching anything.

Read and follow the prompt template in `prompts/00-preflight.md`.

### What it produces

- `INVENTORY.md` — structured snapshot of the codebase:
  - File tree with line counts and detected languages
  - Entry points and data flow summary
  - Dependency graph (which modules import/call which)
  - Tech stack summary (frameworks, language versions, package managers)
- `claudify/DIAGRAMS.md` — all architecture diagrams in a single file (before, after, diff)
- `PROTECTED_PATHS.md` — list of paths that must not be read or modified

### Auto-detection

The pre-flight step auto-detects:

**Project type** by scanning for marker files:
- `requirements.txt`, `pyproject.toml`, `setup.py`, `Pipfile` → Python
- `package.json`, `tsconfig.json` → Node/TypeScript
- Both present → mixed project
- Neither → prompt the user

**Protected paths** by scanning for common sensitive patterns:
- Environment files: `.env`, `.env.*`
- Secret stores: `secrets/`, `credentials/`, `*.pem`, `*.key`
- Raw data directories: `data/raw/`, `db/new/`, `datasets/`
- Large binary directories: files > 10MB
- Anything in `.gitignore` that looks like data or secrets

After auto-detection, present the discovered protected paths to the user and ask them
to confirm or add more before proceeding. This is critical — never skip this step.

---

## Phase 1 — Architect

**Goal**: Produce a detailed, actionable refactoring plan.

Read and follow the prompt template in `prompts/01-architect.md`.

### What it produces

- `PLAN.md` — a step-by-step refactoring plan containing:
  - Target directory structure
  - File moves and renames (with rationale)
  - Module extraction and consolidation instructions
  - Dependency cleanup recommendations
  - Naming convention standardization
  - Which files to delete (with justification)
  - Sequenced implementation steps (ordered to minimize breakage)

### Planning principles

The plan should optimize for:

- **Claude Code readability**: Clear module boundaries, descriptive names, single
  responsibility per file. Claude Code works best when it can understand a module
  without reading the entire codebase.
- **Minimal disruption**: Prefer moves over rewrites. Preserve git history where
  possible (use `git mv`).
- **Incremental safety**: Each step in the plan should leave the codebase in a
  working state. No "big bang" refactors.
- **Data preservation**: Raw data, databases, and research outputs are never
  modified or moved without explicit user approval.

### Model routing

- **Phases 0–1** (Pre-flight, Architect): Use **Opus** for stronger reasoning about
  architecture and system design. These phases are read-heavy and planning-heavy.
- **Phases 2–7** (Implementation through Validation): Use **Sonnet** for execution
  speed and cost efficiency. The plan provides enough structure for Sonnet to follow.

---

## Phase 2 — Refactor

**Goal**: Execute the refactoring plan from Phase 1 step by step.

Read and follow the prompt template in `prompts/02-refactor.md`.

All work happens on a `claudify/refactor` git branch. Each step gets its own commit
for granular rollback. Chesterton's Fence items become TODO comments rather than
silent deletions.

---

## Phase 3 — Cleanup

**Goal**: Remove dead code, empty directories, stale configs, and other cruft.

Read and follow the prompt template in `prompts/03-cleanup.md`.

The model builds a categorized deletion list and presents it to the user for
confirmation before removing anything. Each deletion gets its own commit.

---

## Phase 4 — CLAUDE.md

**Goal**: Generate the project's interface file for Claude Code.

Read and follow the prompt template in `prompts/04-claude-md.md`.

Optimized for Claude Code's context window — concise, specific, exact commands.
Includes protected paths with explanations, coding standards, and domain terminology.

---

## Phase 5 — README.md

**Goal**: Generate human-facing documentation with Mermaid architecture diagrams.

Read and follow the prompt template in `prompts/05-readme.md`.

Adds `## Phase 5 — Architecture After` section to `claudify/DIAGRAMS.md` as the bookend to Phase 0's "before" diagram.

---

## Phase 6 — Tests

**Goal**: Scaffold a test suite targeting the highest-value modules first.

Read and follow the prompt template in `prompts/06-tests.md`.

Prioritizes core business logic and data transformations. Sets up test infrastructure
(conftest.py, fixtures, mocking patterns) for ongoing development.

---

## Phase 7 — Validation

**Goal**: Prove the refactor worked. Reconcile all files, run all checks, produce
a before/after comparison and final report.

Read and follow the prompt template in `prompts/07-validation.md`.

Adds `## Phase 7 — Before vs After Diff` section to `claudify/DIAGRAMS.md` (color-coded comparison) and
writes `claudify/CLAUDIFY_REPORT.md` with metrics, reconciliation table, and remaining items.

---

## Artifacts produced

By the end of all phases, Claudify generates:

**Project root (human/tool facing — stay at root):**

| File | Phase | Purpose |
|------|-------|---------|
| `CLAUDE.md` | 4 | Claude Code project context file |
| `README.md` | 5 | Human-facing documentation |
| `tests/` | 6 | Test suite with infrastructure |

**`claudify/` folder (session artifacts — grouped together):**

| File | Phase | Purpose |
|------|-------|---------|
| `claudify/PROTECTED_PATHS.md` | 0 | Confirmed list of untouchable paths |
| `claudify/INVENTORY.md` | 0 | Pre-refactor codebase snapshot with annotated file tree |
| `claudify/PLAN.md` | 1 | Step-by-step refactoring plan |
| `claudify/REFACTOR_LOG.md` | 2 | Implementation record and deviations |
| `claudify/CLEANUP_LOG.md` | 3 | What was removed and what was kept |
| `claudify/CLAUDIFY_REPORT.md` | 7 | Final validation report with metrics |
| `claudify/DIAGRAMS.md` | 0→7 | All architecture diagrams (before, after, diff) — updated each phase |

---

## Soft dependencies

Claudify works fully standalone, but integrates with these skills when available:

### `/zenify-codebase` — Coding standards

If present, Claudify uses zenify as the authoritative coding standard for all
refactoring recommendations. Its "Type-Rich Minimalism" philosophy covers strict
type hinting, intent-based naming (`verb_noun`, `is_` prefixes), contextual
docstrings, and refactoring guardrails (Chesterton's Fence, flat-over-nested).

If unavailable, the same principles are built directly into Phase 1's Section 5,
so the plan still enforces consistent standards.

### `/mermaid-diagrams` — Diagram generation

If present, Claudify uses it as the guide for all Mermaid diagram creation in
Phase 0 (architecture before), Phase 5 (README diagrams), and Phase 7 (validation
diff diagram). Provides syntax patterns, styling conventions, and diagram type
selection best practices.

If unavailable, built-in diagram guidelines are included in the relevant prompt
templates (Phase 5 has the most detailed fallback).

---

## Prompt template conventions

Each prompt template in `prompts/` follows this structure:

```
# Phase N — [Name]
## Context
Brief description of what this phase does and what it expects.
## Prerequisites
What must exist before running this phase.
## Instructions
The actual prompt content to be run in Claude Code.
## Success criteria
How to verify the phase completed correctly.
```

The user can run these directly or adapt them to their project.

---

## Important behaviors

- **Always confirm protected paths** before reading any files. The auto-detected list
  is a starting point, not gospel.
- **Always generate the "before" diagram** in pre-flight. This gives the user a visual
  baseline to verify consistency after refactoring.
- **Never modify files** in the discovery/planning phases. These phases are read-only.
- **Respect the user's codebase conventions** where they exist. If the project
  already has a clear pattern (even a messy one), note it in the plan rather than
  imposing a completely foreign structure.
- **Surface uncertainties** in the plan. If a file's purpose is unclear, flag it
  rather than guessing and moving it to the wrong place.
- in Mermaid use <br> for newlines , never \n