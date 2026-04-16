# Phase 7 — Post-Refactor Validation & Diff Diagram

## Context

This is the final phase. It validates that the refactored codebase is complete and
consistent, generates a before/after architecture comparison, and produces a summary
report for the user. This is the "prove it worked" step — the user should walk away
confident that nothing was lost and the codebase is genuinely better.

## Prerequisites

- Completed Phases 0–6:
  - `INVENTORY.md` (pre-refactor snapshot)
  - `ARCHITECTURE_BEFORE.mermaid`
  - `PROTECTED_PATHS.md`
  - `PLAN.md`
  - `REFACTOR_LOG.md`
  - `CLEANUP_LOG.md`
  - `CLAUDE.md`
  - `README.md`
  - `ARCHITECTURE_AFTER.mermaid` (from Phase 5)
  - `tests/` directory with passing tests
- All changes on `claudify/refactor` branch
- Claude Code session with **Sonnet** model

## Optional skill dependency

If `/mermaid-diagrams` is available, use it for the diff diagram generation.
If unavailable, follow the built-in guidelines below.

## Instructions

Run the following in Claude Code:

```
Claudify Phase 7: Post-refactor validation and final report.

Read INVENTORY.md, ARCHITECTURE_BEFORE.mermaid, ARCHITECTURE_AFTER.mermaid,
PROTECTED_PATHS.md, PLAN.md, REFACTOR_LOG.md, and CLEANUP_LOG.md.

## Step 1: File accountability check

Every file from the original INVENTORY.md must be accounted for. Build a
reconciliation table:

| Original file | Status | New location | Notes |
|---|---|---|---|
| src/proc.py | Moved + renamed | src/data/process_market_data.py | |
| utils/helpers.py | Split | src/utils/formatting.py, src/utils/validation.py | |
| old_script.py | Deleted | — | Dead code, see CLEANUP_LOG.md |
| .env | Protected | .env (unchanged) | |

Verify:
- No file from INVENTORY.md is unaccounted for (moved, deleted with reason,
  or protected)
- No unexpected new files appeared that aren't documented in REFACTOR_LOG.md
  or CLEANUP_LOG.md
- All protected paths are untouched

If any file is unaccounted for, flag it immediately and ask the user.

## Step 2: Functional validation

Run a comprehensive check that the codebase works:

1. **Import check**: Verify every Python module can be imported without errors:
   ```
   find . -name "*.py" -not -path "./tests/*" -not -path "./.git/*" | while read f; do
     module=$(echo "$f" | sed 's|^\./||;s|/|.|g;s|\.py$||')
     python -c "import $module" 2>&1 || echo "FAIL: $module"
   done
   ```
   (Adapt for Node/TypeScript projects)

2. **Test suite**: Run the full test suite and capture results:
   `pytest tests/ -v --tb=short 2>&1 | tee validation_test_results.txt`

3. **Entry point check**: For each entry point identified in INVENTORY.md,
   verify it can be invoked (with --help or a dry-run flag if available).

4. **Dependency check**: Verify no circular imports were introduced:
   `python -c "import importlib; import pkgutil; ..."` or use a tool like
   `pydeps` or `import-linter` if available.

Report any failures. If everything passes, note it explicitly.

## Step 3: Before/after diff diagram

Add the diff diagram as `## Phase 7 — Before vs After Diff` in `claudify/DIAGRAMS.md`.
Do NOT create a separate `.mermaid` file — all diagrams live in `claudify/DIAGRAMS.md`.

Show both architectures side by side or as a visual diff:

Option A — Side-by-side (preferred for smaller projects):
```mermaid
graph LR
    subgraph Before
        direction TB
        ...original architecture...
    end
    subgraph After
        direction TB
        ...refactored architecture...
    end
```

Option B — Annotated single diagram (preferred for larger projects):
Show the final architecture with color-coded annotations:
- 🟢 Green nodes: new modules (created during refactor)
- 🔵 Blue nodes: moved/renamed modules (existed before, new location)
- 🟡 Yellow nodes: modified modules (same location, code changes)
- ⚪ Gray nodes: unchanged modules
- 🔴 Red dashed nodes: deleted modules (shown as ghosts for reference)

Style example:
```
style NewModule fill:#c8e6c9,stroke:#2e7d32
style MovedModule fill:#bbdefb,stroke:#1565c0
style ModifiedModule fill:#fff9c4,stroke:#f9a825
style UnchangedModule fill:#f5f5f5,stroke:#9e9e9e
style DeletedModule fill:#ffcdd2,stroke:#c62828,stroke-dasharray: 5 5
```

Choose whichever option produces the clearer result for this specific project.

## Step 4: Metrics summary

Calculate and report:

- **File count**: before → after (and net change)
- **Total lines of code**: before → after (excluding tests)
- **Test coverage**: lines/branches covered by the new test suite
- **Module count**: number of importable modules before → after
- **Avg module size**: average lines per module before → after
- **Dependency depth**: longest import chain before → after
- **Protected paths**: confirmed untouched count

These give the user a quantitative sense of what changed.

## Step 5: Final report

Write `CLAUDIFY_REPORT.md` at the project root:

### Summary
One paragraph: what was done, what improved, what to watch out for.

### File reconciliation
The table from Step 1.

### Validation results
- Import check: pass/fail count
- Test suite: pass/fail/skip count
- Entry points: all working / issues found
- Circular dependencies: none / found

### Metrics
The before/after numbers from Step 4.

### Architecture comparison
Embed or link to ARCHITECTURE_DIFF.mermaid.
Also link to ARCHITECTURE_BEFORE.mermaid and ARCHITECTURE_AFTER.mermaid.

### Remaining items
- TODO(claudify) items from REFACTOR_LOG.md that weren't resolved in cleanup
- Chesterton's Fence items still flagged for human review
- Suggested next steps (additional tests, performance profiling, CI setup, etc.)

### Artifacts produced
List all files Claudify generated:

**Project root (human/tool facing):**
- `CLAUDE.md`
- `README.md`
- `tests/`

**`claudify/` folder (session artifacts):**
- `claudify/INVENTORY.md`
- `claudify/PROTECTED_PATHS.md`
- `claudify/PLAN.md`
- `claudify/REFACTOR_LOG.md`
- `claudify/CLEANUP_LOG.md`
- `claudify/CLAUDIFY_REPORT.md`
- `claudify/DIAGRAMS.md` — all architecture diagrams (before, after, diff)

---

Commit everything:
`git add -A && git commit -m "claudify: final validation report and diff diagram"`

Present CLAUDIFY_REPORT.md to the user as the final deliverable. The branch is
ready for their review and merge decision.
```

## Success criteria

After this phase completes, verify:

- [ ] Every file from INVENTORY.md is accounted for in the reconciliation table
- [ ] All imports resolve without errors
- [ ] Test suite passes
- [ ] Entry points work
- [ ] No circular dependencies introduced
- [ ] `claudify/DIAGRAMS.md` contains the Phase 7 diff section
- [ ] `claudify/CLAUDIFY_REPORT.md` exists with all sections
- [ ] Metrics show quantitative before/after comparison
- [ ] Remaining TODO items are documented, not forgotten
- [ ] All Claudify artifacts are listed
- [ ] Everything committed on `claudify/refactor` branch
- [ ] Branch is NOT merged — user makes that decision
