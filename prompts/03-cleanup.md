# Phase 3 — Cleanup Unused Files & Directories

## Context

After the refactor, the codebase may still contain dead code, empty directories,
stale configs, duplicate files, and other cruft that the refactoring plan flagged
but didn't remove (or that became apparent during implementation). This phase
carefully removes them.

This phase modifies files — specifically by deleting them. Every deletion is
justified and committed individually for easy rollback.

## Prerequisites

- Completed Phase 2:
  - All refactoring steps executed on `claudify/refactor` branch
  - `REFACTOR_LOG.md` exists
  - `PLAN.md` Section 4 (Cleanup Candidates) available
- Git working tree is clean on the `claudify/refactor` branch

## Instructions

Run the following in Claude Code:

```
Claudify Phase 3: Cleanup unused files and directories.

Read PLAN.md (Section 4 — Cleanup Candidates), REFACTOR_LOG.md, and PROTECTED_PATHS.md.
Do NOT touch files listed in PROTECTED_PATHS.md.

## Step 1: Build the cleanup list

Start with the candidates from PLAN.md Section 4. Then scan for additional cruft
that may have surfaced during refactoring:

- **Dead code**: Files/functions that are defined but never imported or called.
  Verify by searching for references across the codebase:
  `grep -r "import <module>" . --include="*.py"` (or equivalent for the language)
  `grep -r "<function_name>" . --include="*.py"`
  A file with zero internal importers and no entry point is a candidate.

- **Empty directories**: Left behind after file moves.
  `find . -type d -empty -not -path './.git/*'`

- **Stale configs**: Config files that reference old paths, removed modules,
  or tools no longer in use.

- **Duplicate implementations**: Functions or classes that do the same thing,
  where Phase 2 consolidated them but didn't delete the original.

- **Orphaned __init__.py**: Init files in directories that no longer contain
  Python modules.

- **TODO(claudify) items**: Check REFACTOR_LOG.md for Chesterton's Fence items.
  For each one, determine if it's now safe to remove given the refactored context.
  If still uncertain, leave it and note it in the cleanup log.

Present the full cleanup list to the user and ask for confirmation before
deleting anything. Group by category (dead code, empty dirs, stale configs, etc.)
and include the justification for each deletion.

## Step 2: Execute cleanup

After user confirmation:

For each item in the approved cleanup list:
1. Delete the file or directory
2. Verify nothing breaks:
   - Run syntax/import checks on files that previously referenced the deleted item
   - If tests exist, run them
3. Commit individually:
   `git add -A && git commit -m "claudify: cleanup — remove <item> (<reason>)"`

Order deletions safely:
1. Empty directories first (zero risk)
2. Orphaned __init__.py files second
3. Stale configs third
4. Dead code last (highest risk, most verification needed)

## Step 3: Final scan

After all deletions:

1. Run `find . -type d -empty -not -path './.git/*'` one more time — the deletions
   may have created new empty directories.

2. Verify the full project still works:
   - All entry points importable/runnable
   - Test suite passes (if it exists)

3. Write `CLEANUP_LOG.md`:
   - Files and directories removed (with reason)
   - Items that were candidates but kept (with reason)
   - Any remaining TODO(claudify) items that need human review
   - Total lines of code removed

## Guardrails

- **Never delete without user confirmation.** Present the list, get approval, then act.
- **Never delete protected paths.** Even if they look unused.
- **Never delete test files** unless they test modules that no longer exist AND
  the user explicitly approves.
- **When in doubt, keep it.** A file that might be needed costs nothing to keep.
  A file that was needed and got deleted costs time to recover.
- **Chesterton's Fence still applies.** If a file has edge-case handlers, try/except
  blocks, or defensive logic that you can't fully trace — don't delete it.
```

## Success criteria

After this phase completes, verify:

- [ ] User confirmed the cleanup list before any deletions
- [ ] Each deletion has its own git commit with a descriptive message
- [ ] No protected paths were touched
- [ ] No empty directories remain (except intentionally kept ones)
- [ ] Project entry points still work
- [ ] Tests still pass (if they exist)
- [ ] `CLEANUP_LOG.md` exists documenting what was removed and what was kept
- [ ] No Chesterton's Fence items were deleted without resolution
