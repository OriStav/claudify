# Phase 6 — Test Scaffolding

## Context

The refactored codebase needs tests to protect against regressions in future work.
This phase doesn't aim for 100% coverage — it targets the highest-value tests first:
core business logic, data transformations, and entry points. The goal is a test suite
that a developer (or Claude Code) can run after any change to confirm nothing broke.

Research codebases often have zero tests. This phase creates the foundation.

## Prerequisites

- Completed Phases 0–5:
  - `CLAUDE.md` (for understanding project purpose and key modules)
  - `PLAN.md` (Section 3 for module relationships)
  - `INVENTORY.md` (for identifying core vs peripheral modules)
  - Refactored codebase on `claudify/refactor` branch
- Claude Code session with **Sonnet** model

## Instructions

Run the following in Claude Code:

```
Claudify Phase 6: Create test scaffolding for the codebase.

Read CLAUDE.md (Sections 2, 3, 4), PLAN.md (Section 2 — Target Structure),
and PROTECTED_PATHS.md. Scan the current codebase to understand module boundaries.

## Step 1: Identify what to test

Prioritize tests by value. Not everything needs a test — focus on what matters most:

**Priority 1 — Core logic (must test):**
- Functions that transform data (parsing, calculations, aggregations)
- Business logic with conditional paths
- Functions that other modules depend on heavily (high reverse-dep count)

**Priority 2 — Integration points (should test):**
- Entry points (CLI commands, main scripts)
- Data pipeline stages (input → transform → output)
- External API interaction wrappers (test the wrapper logic, mock the API)

**Priority 3 — Utilities (nice to have):**
- Helper functions with non-trivial logic
- Config loading and validation
- Type conversion and formatting functions

**Skip:**
- Pure I/O with no logic (just reads a file and returns it)
- Simple getters/setters
- Anything in PROTECTED_PATHS.md
- Generated code or vendored dependencies

Present the prioritized list to the user and ask if they want to adjust the
scope before writing tests.

## Step 2: Set up test infrastructure

Based on the project type:

**Python:**
- Create `tests/` directory at project root
- Create `tests/__init__.py`
- Create `tests/conftest.py` with shared fixtures:
  - Sample data fixtures for the project's domain
  - Temp directory fixtures for file I/O tests
  - Mock fixtures for external APIs/services
- Ensure pytest is in requirements (add to requirements.txt or pyproject.toml
  if missing)
- Create a `pytest.ini` or `[tool.pytest.ini_options]` section with:
  - Test discovery paths
  - Sensible defaults (verbose output, short traceback)

**Node/TypeScript:**
- Create `tests/` or `__tests__/` directory
- Configure test runner (Jest or Vitest) in package.json
- Create test utilities file with shared helpers and mocks

## Step 3: Write the tests

For each module in the prioritized list:

1. Create a test file: `tests/test_<module_name>.py` (Python) or
   `tests/<module_name>.test.ts` (TypeScript)

2. Structure each test file:
   - Group related tests in classes or describe blocks
   - Name tests descriptively: `test_calculate_order_total_returns_zero_for_inactive_order`
   - Follow Arrange-Act-Assert pattern

3. For each function being tested, write:
   - **Happy path**: Normal input → expected output
   - **Edge cases**: Empty input, None/null, boundary values
   - **Error cases**: Invalid input → expected error or graceful handling

4. Mocking strategy:
   - Mock external APIs and services — never make real network calls in tests
   - Mock file system operations that touch protected paths
   - Use real objects for internal module interactions (integration-style)
   - Keep mocks minimal — over-mocking makes tests brittle

5. Test data:
   - Use realistic sample data from the project's domain
   - Create fixtures in conftest.py for reusable test data
   - Never use data from protected paths — create synthetic equivalents
   - If the project processes specific formats (CSV, JSON, API responses),
     create small sample files in `tests/fixtures/`

## Step 4: Verify and document

1. Run the full test suite and fix any failures:
   - Python: `pytest tests/ -v`
   - Node: `npm test`

2. Add a coverage check (informational, not a gate):
   - Python: `pytest tests/ --cov=<source_dir> --cov-report=term-missing`
   - Note which Priority 1 modules have coverage and which don't

3. Update CLAUDE.md Section 4 (Getting Started) to include the test command.

4. Commit:
   `git add tests/ pytest.ini requirements.txt CLAUDE.md && git commit -m "claudify: add test scaffolding with Priority 1 coverage"`

## Test writing standards

Follow Type-Rich Minimalism in test code too:
- Type hints on test helper functions and fixtures
- Descriptive test names (verb_noun, describing the scenario)
- Docstrings on test classes explaining what module/behavior they cover
- No magic numbers — use named constants or fixtures
```

## Success criteria

After this phase completes, verify:

- [ ] `tests/` directory exists with proper structure
- [ ] Test infrastructure is configured (conftest.py/fixtures, pytest.ini/jest.config)
- [ ] All Priority 1 modules (core logic) have at least one test file
- [ ] Tests include happy path, edge cases, and error cases
- [ ] External dependencies are mocked — no real network calls
- [ ] No test reads from protected paths
- [ ] Test suite runs and passes: `pytest tests/ -v` or `npm test`
- [ ] CLAUDE.md updated with test commands
- [ ] Committed on the `claudify/refactor` branch
