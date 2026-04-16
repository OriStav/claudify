# Phase 2 — Refactor Implementation

## Context

This phase executes the refactoring plan produced in Phase 1. Each step in PLAN.md
is designed to be atomic — completable in a single prompt — and to leave the codebase
in a working state. This is where the actual file moves, renames, and code changes happen.

This is the first phase that modifies files.

## Prerequisites

- Completed Phase 0 and Phase 1 outputs:
  - `INVENTORY.md`
  - `ARCHITECTURE_BEFORE.mermaid`
  - `PROTECTED_PATHS.md`
  - `PLAN.md`
- Claude Code session with **Sonnet** model (recommended for implementation speed)
- Git repository in a clean state (`git status` shows no uncommitted changes)

## Instructions

Run the following in Claude Code:

```
Claudify Phase 2: Implement the refactoring plan.

Read PLAN.md, PROTECTED_PATHS.md, and ARCHITECTURE_BEFORE.mermaid.
Do NOT read or modify files listed in PROTECTED_PATHS.md.

## Before starting

1. Verify git is clean:
   Run `git status` — if there are uncommitted changes, stop and ask the user
   to commit or stash them first. We need a clean baseline to roll back if needed.

2. Create a dedicated branch:
   Run `git checkout -b claudify/refactor`
   All changes happen on this branch. The user merges when satisfied.

3. Read PLAN.md fully before touching any files. Understand the full sequence
   so you don't make a change in step 3 that conflicts with step 7.

## Implementation loop

For each numbered step in PLAN.md Section 3 (Implementation Steps), do the following:

### Execute the step
- Follow the step's instructions exactly (file moves, renames, code changes)
- Use `git mv` for moves/renames to preserve history
- When creating new directories, use `mkdir -p`
- Apply Type-Rich Minimalism standards as specified in the step:
  - Add type hints where the step calls for them
  - Rename variables/functions to match intent-based naming
  - Add contextual docstrings where specified
  - Flatten nested logic using guard clauses where specified
- Update all import paths and references affected by the change

### Verify the step
- Run the verification check specified in the step (e.g., "confirm imports resolve",
  "run tests", "python -c 'import module'")
- If the step doesn't specify a check, at minimum verify:
  - No syntax errors in modified files (`python -m py_compile <file>` for Python)
  - Import chains still resolve
- If verification fails, fix the issue before moving to the next step

### Commit the step
- **Batch small steps into grouped commits** to keep history readable:
  - Steps that are purely scaffolding (mkdir, __init__.py) → one commit
  - Steps that move a logical group of files together (e.g., all pipeline modules) → one commit
  - Steps with non-trivial code changes (import rewrites, logic edits) → one commit each
  - Never batch more than ~10 files or 2 phases of work into a single commit
- Commit message format: `refactor(steps N-M): <brief description>`
  Example: `refactor(steps 16-24): move pipeline modules from methods/ to pipeline/`

## Guardrails during implementation

- **Chesterton's Fence**: If you encounter logic that PLAN.md marks for removal but
  you're uncertain about — keep it and add a TODO comment instead:
  `# TODO(claudify): Review — possibly redundant, see PLAN.md step N`
- **Protected paths**: If a step's changes would affect files near protected paths
  (e.g., updating an import that references a protected module), stop and ask the
  user before proceeding.
- **Scope discipline**: Only make changes described in the current step. Do not
  "fix ahead" or refactor things you notice that aren't in the plan. Note them
  for later instead.
- **Preserve behavior**: The goal is to reorganize, not rewrite. If a function
  works (even if ugly), move it faithfully first. Code quality improvements happen
  within the step only where PLAN.md explicitly calls for them.

## After all steps complete

1. Run a full verification:
   - If tests exist: run the test suite
   - If no tests: verify all entry points can be imported/executed without errors
   - Check that no files from INVENTORY.md were accidentally lost

2. Write `claudify/REFACTOR_LOG.md` summarizing:
   - How many steps were executed
   - Any steps that required deviation from the plan (and why)
   - Any TODO comments added (Chesterton's Fence items)
   - Any issues encountered and how they were resolved

3. Do NOT merge the branch. Leave that decision to the user.
```

## Success criteria

After this phase completes, verify:

- [ ] All steps from PLAN.md Section 3 were executed
- [ ] Each step has its own git commit on the `claudify/refactor` branch
- [ ] No files in PROTECTED_PATHS.md were read or modified
- [ ] All modified files pass syntax checks
- [ ] Import chains resolve correctly
- [ ] Entry points still work (or tests pass if they exist)
- [ ] `REFACTOR_LOG.md` exists documenting the implementation
- [ ] Type-Rich Minimalism standards were applied where PLAN.md specified
- [ ] The branch has NOT been merged (user decides)
