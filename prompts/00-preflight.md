# Phase 0 — Pre-flight Inventory

## Context

This phase scans the codebase to build a complete inventory before any changes are made.
It produces a structured snapshot, architecture diagram, and confirmed list of protected
paths. Everything here is read-only — no files are modified.

## Prerequisites

- A codebase directory to analyze
- Claude Code session with Opus model (recommended)

## Instructions

Run the following in Claude Code (adapt the path to your project):

```
Claudify Phase 0: Pre-flight inventory for this codebase.

1. **Detect project type**:
   Scan for marker files (requirements.txt, package.json, pyproject.toml, tsconfig.json,
   Pipfile, setup.py, Cargo.toml, go.mod) and report what kind of project this is.

2. **Detect protected paths**:
   Scan for files and directories that should NOT be read or modified:
   - .env, .env.*, secrets/, credentials/, *.pem, *.key
   - Data directories: data/raw/, db/new/, datasets/, *.sqlite, *.db
   - Large files (>10MB)
   - Patterns from .gitignore that look like data or secrets
   - __pycache__/, node_modules/, .git/
   Present the discovered list and ASK ME to confirm or add more before proceeding.
   Write the confirmed list to PROTECTED_PATHS.md.

3. **Build file inventory**:
   For every file NOT in the protected paths list, catalog:
   - Relative path
   - Language (detected from extension)
   - Line count
   - Brief purpose (from first docstring/comment, or inferred from name)
   Group by directory. Include a summary table at the top.

4. **Map entry points and data flow**:
   Identify:
   - Scripts meant to be run directly (if __name__ == "__main__", bin/ scripts, CLI entry points)
   - Main application entry points (app.py, index.js, main.*)
   - Config files that control behavior
   - Data flow: where does data come in, how is it transformed, where does it go out?
   Summarize as a narrative, not just a list.

5. **Map dependency graph**:
   For each non-trivial module, list:
   - What it imports from other project modules (internal deps)
   - What it imports from external packages (external deps)
   - What other modules import it (reverse deps)
   Identify clusters of tightly coupled modules and any orphan files with no importers.

6. **Generate architecture diagram**:
   Add the "before" architecture diagram as the first section of `claudify/DIAGRAMS.md`.
   Use a `## Phase 0 — Architecture Before` heading, then a ```mermaid block showing:
   - Module groupings / directories as subgraphs
   - Import relationships as arrows
   - Entry points highlighted
   - Data flow direction
   Keep it readable — collapse trivial utility modules into their parent group.

7. **Write INVENTORY.md**:
   Create `claudify/INVENTORY.md` — all Claudify artifacts go in the `claudify/` folder.
   Combine all of the above into a single structured document:
   - Tech stack summary (top)
   - **Annotated file tree** using this style (inline comments on each file/dir):
     ```
     project/
     ├── app.py                  # Entry point — wires sidebar + tabs (thin orchestrator)
     ├── paths.py                # File path registry
     │
     ├── core/                   # Business logic — no Streamlit imports
     │   ├── calendar_api.py     # Google Calendar API + CSV cache (24 h TTL)
     │   ├── config.py           # SCORE_DCT, budgets, lazy calendar loader
     │   └── scoring.py          # Event pipeline, reward calc, period delta
     ```
     One concise comment per file describing its actual purpose. No line counts — just purpose.
   - Entry points and data flow narrative
   - Dependency graph summary
   - Link to `claudify/DIAGRAMS.md` for the architecture diagram
   - Protected paths reference

Do NOT read or catalog any files in the protected paths. Do NOT modify any files.
```

## Success criteria

After this phase completes, verify:

- [ ] `claudify/PROTECTED_PATHS.md` exists and was confirmed by the user
- [ ] `claudify/INVENTORY.md` contains an annotated file tree, entry points, dependency graph, and tech stack
- [ ] `claudify/DIAGRAMS.md` contains the Phase 0 "Architecture Before" diagram
- [ ] No files in the codebase were modified
- [ ] The user can look at the mermaid diagram and say "yes, that's my codebase"
