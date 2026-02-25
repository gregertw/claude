# Iterate on Implementation

Iterate on an already-implemented plan: fix bugs, refine UX, add small enhancements, and continuously update the plan document to track what changed and why.

Use this command when you've implemented a plan (via `/implement_plan`) and verified it (via `/verify_implementation`), but still need to iterate on details — fixing issues found during testing, refining flow and wording, tweaking specific functionality, etc.

## Arguments

$ARGUMENTS - Optional: path to the plan file, or description of what to iterate on

## Process

### 1. Load context

- If a plan path was provided as argument, read it fully
- If the plan has not been implemented or not started, stop and tell the user to user /update_plan to make changes to a plan that has not been started
- If not, list recent files in `thoughts/plans/` and ask which plan is being iterated
- Read the plan document and the verification document (if referenced)
- Check the verification's "Remaining Tasks" and "Issues Found" sections for known work items
- Run `git diff --stat HEAD` to understand the current state of uncommitted changes

### 2. Understand the iteration scope

Ask the user what needs to change. Iteration typically falls into these categories:

- **Bug fixes**: Something doesn't work as expected
- **UX refinements**: Flow, wording, visual tweaks, interaction details
- **Missing functionality**: Small features not covered by the plan (e.g., a back-link, a loading state)
- **Verification fixes**: Issues identified by `/verify_implementation`
- **Integration adjustments**: Components need different props, APIs need different contracts

For each item, confirm with the user:
1. What specifically needs to change?
2. Which files are affected?
3. Is this a deviation from the plan, or within the plan's intent?

### 3. Implement the change

For each change:

**Before coding:**
- Do not go into plan mode
- Read all files that will be modified
- Understand how the change fits with existing code

**Make the change:**
- Implement the fix/refinement
- Run relevant tests to verify

**After each change, immediately update the plan document:**

Add or update the **Post-Verification Changes** section in the plan. If the section doesn't exist, add it before "Implementation Summary" with this structure:

```markdown
## Post-Verification Changes (YYYY-MM-DD)

After the initial implementation was verified (commit `XXXXXX`, verification in `thoughts/verifications/...`), the following changes were made during iteration.

### N. [Short title describing the change]

**Category**: [Bug fix | UX refinement | Decision changed | New functionality | Verification fix]

**What changed**: [Specific description of the change]

**Files affected**:
- `path/to/file.tsx` — [what changed in this file]

**Rationale**: [Why this change was needed — what wasn't working, what was discovered]
```

If the section already exists, append new entries with incrementing numbers.

### 4. Update related plan sections

If the change modifies a decision, phase behavior, or component contract:

- **Decisions Made**: Add `[Updated YYYY-MM-DD]` annotation to changed decisions, or add new decisions
- **Phase N sections**: Update the "Deviations" subsection with what changed
- **Implementation Summary > Deviations from Plan**: Add the deviation
- **Implementation Summary > Learnings**: Add any new insights

### 5. Run verification after each logical group of changes

After completing a coherent set of changes (e.g., all UX fixes, or all bug fixes), run the necessary verifications as specified in CLAUDE:md, e.g. for a python code repository:

```bash
poetry run ruff check . --fix
poetry run pyright
poetry run pytest
npm --prefix frontend run build
```

Fix any failures before moving to the next group.

### 6. Summarize iteration progress

After all changes are complete, present to the user:

```
## Iteration Summary

### Changes Made
1. [Change title] — [one-line description]
2. [Change title] — [one-line description]

### Plan Updated
- Post-Verification Changes section: [N new entries]
- Decisions updated: [list any]
- Phases affected: [list any]

### Verification Status
- ruff: [pass/fail]
- pyright: [pass/fail]
- pytest: [pass/fail]
- frontend build: [pass/fail]
```

## Guidelines

- **Update the plan after EVERY change, not at the end** — this is the whole point. The plan document should always reflect reality.
- **Small, focused changes** — each iteration item should be independently testable and documentable. Don't batch unrelated changes.
- **Preserve the plan's structure** — add to the Post-Verification Changes section, don't reorganize existing phases.
- **Track decisions explicitly** — if you're changing how something works vs what the plan specified, that's a decision change. Document why.
- **Track bugs explicitly** — if you find and fix a bug during iteration, document what the bug was and how it was fixed, not just the fix.
- **Be specific about files** — always list which files were modified and what changed in each.
- **Don't over-document** — a bug fix that changes one line doesn't need a paragraph. Match documentation depth to change significance.
- **Run tests frequently** — don't accumulate 10 changes before running tests. Test after each coherent group.
