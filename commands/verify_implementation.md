# Verify Implementation

Review the implementation against the plan and research documents. Verify correctness, test coverage, and create a verification document with findings.

## Where the document goes, and what it does to the plan

The verification goes in `thoughts/verifications/YYYY-MM-DD-slug.md`, **reusing
the plan's slug** — same slug across research → plan → verification is how a
piece of work is followed end to end. The date is today's, not the plan's.

The document is a dated snapshot: it records what was true when it was written
and is not edited afterwards. It carries **no `status:` frontmatter** — only
plans do.

Two things this command owns on the plan side:

- **Write the back-link.** When the verification confirms the plan landed, set
  the plan's frontmatter to `status: done` and add
  `verified: thoughts/verifications/YYYY-MM-DD-slug.md`.
- **Only link what you actually verified.** `verified:` asserts that *this plan*
  was verified. If the verification covers a subset — two follow-up items, one
  phase of six — say so in the document and leave the plan at bare
  `status: done`; the verification's own `**Plan:**` line keeps the thread
  discoverable. An over-claimed `verified:` reads as proof later.

**Never move the plan** on the strength of a verification, and never create a
`completed/` directory: a directory is a *kind* of document, never a *status*.
The `**Plan:**` path you write below is exactly the link a move would rot.

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

Save to `thoughts/verifications/YYYY-MM-DD-slug.md`:

```markdown
# Verification: [Feature Name]

**Date:** YYYY-MM-DD
**Plan:** thoughts/plans/YYYY-MM-DD-slug.md
**Research:** thoughts/research/YYYY-MM-DD-slug.md
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

### 6. Update the plan

- Set the plan's frontmatter `status: done` and add `verified:` pointing at the
  document you just wrote — unless the verification only covers a subset of the
  plan, in which case leave `verified:` off and say why in the document
- Leave the plan file where it is
- Anything in "Remaining Tasks" that nobody is picking up now belongs in
  `thoughts/todo/<slug>.md`, so the plan can close instead of lingering

### 7. Present findings

Show the user a summary of the verification with the document path, highlighting any issues that need attention.

## Guidelines

- **Read the actual code** - don't rely on the plan's claims of completion
- **Run all checks yourself** - verify automated results firsthand
- **Be specific** about issues - include file paths, line numbers, and concrete descriptions
- **Distinguish severity** - not every deviation is a problem
- **Check test quality** - tests that exist but don't test the right things are worse than no tests
