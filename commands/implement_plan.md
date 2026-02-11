# Implement Plan

Execute an implementation plan phase by phase, ensuring tests pass at each step and keeping the plan document updated with progress and learnings.

## Process

### 1. Load the plan

- If a plan path was provided as argument, read it fully
- If not, list recent files in `thoughts/plans/` and ask which to implement
- Read the plan completely, noting any phases already marked as done
- Read the referenced research document and all files mentioned in the plan

### 2. Resume from current state

- Look for phases marked with `Implementation Status: Complete` - skip those
- Start from the first phase marked `Not Started` or `In Progress`
- If resuming a partially completed phase, verify what's already done before continuing

### 3. Implement each phase

For each phase:

**Before coding:**
- Read all files that will be modified
- Understand the current state of the code
- Update the plan: change status to `In Progress`

**Implement the changes:**
- Make the code changes specified in the plan
- Write new tests as specified
- If the plan doesn't match reality (code has changed, approach won't work), STOP:

```
Issue in Phase [N]:
Expected: [what the plan says]
Found: [actual situation]
Proposed adjustment: [how to proceed]

Should I continue with this adjustment?
```

**After coding:**
- Run verification commands for the phase as specified in CLAUDE.md. Example verifications for a python repository:
  ```
  poetry run ruff check . --fix
  poetry run pyright
  poetry run pytest
  ```
- Fix any failures before proceeding
- Update the plan document:
  - Mark the phase: `Implementation Status: Complete`
  - Add notes about any deviations or learnings under the phase
  - If a decision changed during implementation, update "Decisions Made"

### 4. Between phases

After completing each phase:
- Verify ALL tests still pass (not just the phase's tests)
- Update the plan with any new insights that affect upcoming phases
- Proceed to the next phase

### 5. After all phases complete

- Run the full verification suite one final time
- Update the plan's top-level status to `Implemented`
- Add a summary section at the bottom:

```markdown
## Implementation Summary

**Completed:** YYYY-MM-DD
**All phases:** Complete
**Test status:** All passing

### Deviations from Plan
- [Any changes made during implementation]

### Learnings
- [Anything discovered that wasn't in the plan]
```

- Present the results to the user

## Guidelines

- **Follow the plan's intent** but adapt to what you find in the code
- **Tests must pass after every phase** - never leave things broken between phases
- **Update the plan as you go** - it should always reflect current reality
- **Stop and ask** when something doesn't match the plan significantly
- **Run all checks** (e.g. ruff, pyright, pytest or similar) after each phase, not just tests
- **Read files fully** before modifying them
- **Commit-worthy phases** - each completed phase should be in a state worthy of committing
