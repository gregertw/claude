# Create Implementation Plan

Take a research document, interact with the user to resolve decisions, and produce an actionable implementation plan. Evaluate the plan from multiple perspectives before finalizing.

## Process

### 1. Load context

- If a research doc path was provided as argument, read it fully
- If not, list recent files in `thoughts/research/` and ask which to use
- If no research doc exists, suggest running `/research_codebase` first
- Read all files referenced in the research document

### 2. Resolve open decisions

Walk through each item in the research doc's "Decisions Needed" section:
- Present the options with their trade-offs
- Ask the user to choose
- Record the decision and rationale
- If a decision triggers new questions, research them before continuing

Do NOT proceed to planning until all decisions are resolved.

### 3. Design the implementation approach

Present a phased outline to the user BEFORE writing details:

```
## Proposed phases:
1. [Phase name] - [what it accomplishes]
2. [Phase name] - [what it accomplishes]
3. [Phase name] - [what it accomplishes]

Does this phasing make sense?
```

Get feedback and adjust before writing the full plan.

### 4. Evaluate the plan

Spawn an agent team to evaluate the plan from four perspectives in parallel:

- **Architecture**: Does this fit existing patterns in the codebase? Integration risks? Dependencies?
- **Security**: Authentication, authorization, injection, or data exposure risks?
- **Scalability**: Performance under load? Database query patterns? Memory usage?
- **Usability**: Is the API/UI intuitive? Error messages helpful? Migration path clear?

Each evaluator should review the proposed changes against the actual codebase and return specific concerns with `file:line` references.

Incorporate feedback into the plan. Present any significant concerns to the user.

### 5. Write the plan

Save to `thoughts/plans/YYYY-MM-DD-description.md`:

```markdown
# Implementation Plan: [Feature Name]

**Date:** YYYY-MM-DD
**Status:** Ready for Implementation
**Research:** thoughts/research/YYYY-MM-DD-description.md
**Branch:** [current branch]

## Overview

[What we're building and why, 2-3 sentences]

## Decisions Made

- **[Decision 1]**: [Choice] - [rationale]
- **[Decision 2]**: [Choice] - [rationale]

## What We're NOT Doing

- [Explicit out-of-scope item]
- [Another out-of-scope item]

## Phase 1: [Descriptive Name]

### Changes

- `path/to/file.py` - [what changes and why]
- `path/to/other.py` - [what changes and why]

### New Tests, both unit and integration tests

- [Test description and what it verifies]
- [Another test]

### Verification

- Use information from CLAUDE.md to understand how to run verification, these are examples for a python code base:
    - [ ] `poetry run pytest tests/path` passes
    - [ ] `poetry run ruff check .` passes
    - [ ] `poetry run pyright` passes
    - [ ] [manual verification step if needed]

### Implementation Status: Not Started

---

## Phase 2: [Descriptive Name]

[Same structure...]

---

## Evaluation Notes

### Architecture
[Key feedback and how it was addressed]

### Security
[Key feedback and how it was addressed]

### Scalability
[Key feedback and how it was addressed]

### Usability
[Key feedback and how it was addressed]
```

### 6. Present for approval

Show the user the plan location and a summary. Ask if they want to adjust anything before implementation.

## Guidelines

- **Each phase must be independently testable** - don't create phases that leave things broken
- **Include specific file paths** for all planned changes
- **Every phase needs new tests** for new functionality AND passing existing tests
- **No open questions in the final plan** - resolve everything with the user first
- **Separate automated verification** (commands to run) from manual verification
- **Keep phases small** - prefer more small phases over fewer large ones
