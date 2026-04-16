# Phase 1 — Architectural Review & Plan

## Context

This phase reviews the codebase inventory and architecture diagram from Phase 0, then
produces a detailed, sequenced refactoring plan. The plan is written so that a coding
model (Sonnet) or a human developer can follow it step-by-step without needing to
re-analyze the entire codebase.

This phase is still read-only — no files are modified.

## Prerequisites

- Completed Phase 0 outputs:
  - `INVENTORY.md`
  - `ARCHITECTURE_BEFORE.mermaid`
  - `PROTECTED_PATHS.md`
- Claude Code session with Opus model (recommended for architectural reasoning)

## Optional skill dependency

If available in the user's Claude Code environment, this phase benefits from:

- **`/zenify-codebase`** — Enforces "Type-Rich Minimalism" coding standards during
  the refactoring plan. If available, the plan will reference zenify conventions
  for naming, typing, and docstrings. If unavailable, the core principles are
  built into Section 5 below.

## Instructions

Run the following in Claude Code:

```
Claudify Phase 1: Architectural review and refactoring plan.

Read INVENTORY.md, ARCHITECTURE_BEFORE.mermaid, and PROTECTED_PATHS.md.
Do NOT read files listed in PROTECTED_PATHS.md.

## Optional tooling

If /zenify-codebase is available:
  - Use it as the authoritative coding standard for all refactoring recommendations.
  - Reference its rules directly in the plan's naming and typing sections.
If unavailable, apply the built-in "Type-Rich Minimalism" principles in Section 5.

---

Based on the inventory (and optional tooling output), produce PLAN.md with these sections:

## 1. Assessment

Summarize the current state:
- What patterns exist (even messy ones worth preserving)?
- What are the main pain points (tangled deps, unclear naming, god files, dead code)?
- What's the "conceptual architecture" buried under the mess?
- Where does the code violate Type-Rich Minimalism? (missing types, ambiguous names,
  no docstrings, deeply nested logic)

## 2. Target structure

Propose the target directory layout. For each directory/module:
- Name and purpose
- What files move there (current path → new path)
- Rationale for the grouping

Design the structure for Claude Code effectiveness:
- Each module should be understandable without reading the whole repo
- File names should describe their contents (not "utils.py" or "helpers.js")
- Group by domain/feature, not by file type
- Keep module boundaries clean — minimize cross-cutting imports

## 3. Implementation steps

Write a numbered sequence of steps. Each step must:
- Be atomic (completable in one prompt to Sonnet)
- Leave the codebase in a working state when done
- Specify exactly which files to move, rename, or modify
- Note any import paths or references that need updating
- Flag if the step touches anything near protected paths
- Include a verification check (e.g., "run tests", "confirm imports resolve")

Order the steps to minimize cascading breakage:
1. Leaf modules first (no internal importers)
2. Shared utilities second
3. Core modules third
4. Entry points last

## 4. Cleanup candidates

List files/directories recommended for deletion:
- Dead code (defined but never imported or called)
- Duplicate implementations
- Stale config files
- Empty __init__.py or placeholder files

For each, state why it's safe to delete and what to verify first.

Apply Chesterton's Fence: if a file contains guard clauses, error handlers, or
edge-case logic that seems redundant — flag it for review, don't mark it for deletion.
There's usually a reason it exists.

## 5. Coding standards — Type-Rich Minimalism

Apply these standards to all refactoring recommendations in the plan. The philosophy:
"Readability for Humans, Explicit Context for Machines."

**Type contracts:**
- All function arguments and return values must have type hints
- Use specific types (List[int], Optional[str], Dict[str, Any]) — avoid bare `Any`
- Use dataclasses or Pydantic models for complex objects instead of raw dicts
- Multi-line signatures: closing paren indented one level from `def`, not at column 0

**Intent-based naming:**
- Functions: verb_noun structure (calculate_net_revenue, not calc)
- Variables: no single letters except i/x/y in math/loops
- Booleans: prefix with is_, has_, should_

**Contextual docstrings:**
- Every public function/class gets a docstring explaining the business logic and *why*
- Google Style or simple Markdown format
- Don't describe step-by-step code logic — the code does that

**Refactoring guardrails:**
- Chesterton's Fence: never remove a check, guard clause, or error handler unless
  you can prove it's redundant
- Preserve edge-case handlers explicitly (if x is None or x == "")
- Flat over nested: prefer early returns (guard clauses) over deep if/else

For each step in Section 3, note which of these standards apply and what specific
changes to make (e.g., "rename `proc_data` → `process_market_data`, add type hints
to all 4 functions, add docstrings explaining the CKAN API interaction").

## 6. Naming conventions

If the codebase has inconsistent naming, propose a standard aligned with
Type-Rich Minimalism:
- File naming: snake_case for Python, kebab-case for configs
- Function/class naming: verb_noun functions, PascalCase classes
- Directory naming: lowercase, descriptive, domain-grouped
- Config file conventions

## 7. CLAUDE.md recommendations

Note what should go in the project's CLAUDE.md to help future Claude Code sessions:
- Project purpose and domain context
- Key architectural decisions
- How to run/test the project
- Protected paths and why they're protected
- Domain-specific terminology
- Coding standards reference (link to zenify if available, or embed the core rules)

## 8. Post-refactor architecture

Describe (in prose) what ARCHITECTURE_AFTER.mermaid should look like once the plan
is fully implemented. This gives the user a clear vision of the end state.

---

Write the plan for a Sonnet-class model to execute. That means:
- Be explicit, not implicit. Don't say "reorganize the utils" — say which files go where.
- Include the exact commands where possible (git mv, mkdir, etc.)
- Reference files by their current path AND their target path.
- Keep each step focused — one concern per step.
- For each file being refactored, specify exactly which Type-Rich Minimalism rules apply.
```

## Success criteria

After this phase completes, verify:

- [ ] `PLAN.md` exists with all 8 sections
- [ ] Every file from `INVENTORY.md` is accounted for (moved, kept, or flagged for deletion)
- [ ] Implementation steps are ordered and each is atomic
- [ ] Protected paths are never touched in the plan
- [ ] Type-Rich Minimalism standards are referenced in concrete refactoring steps
- [ ] Chesterton's Fence is applied — no deletions without justification
- [ ] The plan is specific enough that someone unfamiliar with the codebase could follow it
- [ ] No files in the codebase were modified

---

## ⚡ Time to switch models!

> 🧠 **Phases 0–1 used Opus** for architectural reasoning.
> ⚡ **Phases 2–7 use Sonnet** for implementation speed and cost efficiency.
>
> 👉 **Run `/model` now** to switch to Sonnet before starting Phase 2.
> The plan is detailed enough that Sonnet can execute it precisely.
