# Verify Implementation

Review the implementation against the plan and research documents. Verify correctness, test coverage, and create a verification document with findings.

## Process

### 1. Gather context

- If a plan path was provided as argument, read it fully
- If not, list recent files in `thoughts/plans/` and ask which to verify
- Read the referenced research document
- Read the plan, noting which phases are marked complete

### 2. Run all automated checks

Identify which checks to run and how to run tests from CLAUDE.md. Example for a python repository:
```
poetry run ruff check .
poetry run pyright
poetry run pytest
```

Record the results. If anything fails, note it but continue the review.

### 3. Verify each phase

For each phase marked as complete in the plan:

- Read every file listed in the phase's "Changes" section
- Verify the changes match what was planned
- Check that the tests listed in "New Tests" exist and are meaningful
- Run the phase-specific verification commands
- Note any deviations (acceptable or concerning)

Use **Explore** or **codebase-analyzer** sub-agents for parallel investigation when reviewing multiple independent areas.

### 4. Check for gaps

- Are there edge cases not covered by tests?
- Are error paths handled properly?
- Were any planned items skipped without explanation?
- Does the implementation introduce any security concerns?
- Are there leftover TODOs, debug code, excessive logging, or commented-out code?

### 5. Write the verification document

Save to `thoughts/verifications/YYYY-MM-DD-description.md`:

```markdown
# Verification: [Feature Name]

**Date:** YYYY-MM-DD
**Plan:** thoughts/plans/YYYY-MM-DD-description.md
**Research:** thoughts/research/YYYY-MM-DD-description.md
**Branch:** [current branch]
**Commit:** [current short hash]

## Automated Check Results

For each check, specify the check, whether it passed or failed and details if fail. Examples for a python repository:

- **Ruff:** [Pass/Fail - details if fail]
- **Pyright:** [Pass/Fail - details if fail]
- **Pytest:** [Pass/Fail - N passed, N failed, N skipped]

## Phase Verification

### Phase 1: [Name] - [VERIFIED / ISSUES FOUND]

**Changes verified:**
- `file.py:123` - [matches plan / deviation noted]

**Tests verified:**
- [test name] - [passes / meaningful coverage]

**Deviations from plan:**
- [Any differences and whether they're acceptable]

### Phase 2: [Name] - [VERIFIED / ISSUES FOUND]

...

## Remaining Tasks

- [ ] [Anything not yet completed from the plan]
- [ ] [Any follow-up work identified during verification]

## Issues Found

### [Issue 1]
**Severity:** [High/Medium/Low]
**Description:** [What's wrong]
**Location:** `file.py:line`
**Recommendation:** [How to fix]

## Overall Assessment

[1-2 paragraph summary: Is the implementation complete and correct? What needs attention before merging?]
```

### 6. Present findings

Show the user a summary of the verification with the document path, highlighting any issues that need attention.

## Guidelines

- **Read the actual code** - don't rely on the plan's claims of completion
- **Run all checks yourself** - verify automated results firsthand
- **Be specific** about issues - include file paths, line numbers, and concrete descriptions
- **Distinguish severity** - not every deviation is a problem
- **Check test quality** - tests that exist but don't test the right things are worse than no tests
