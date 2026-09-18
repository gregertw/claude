# Create Implementation Plan

Take a research document, interact with the user to resolve decisions, and produce an actionable implementation plan. Evaluate the plan from multiple perspectives before finalizing.

## Where documents live

The layout of `thoughts/` is defined by the project's `thoughts/README.md`,
which is authoritative; the default it starts from is
`~/.claude/templates/thoughts-README.md`: `features/`, `research/`, `plans/`,
`verifications/` (dated snapshots) and `reference/`, `todo/` (living). **A
directory is a *kind* of document, never a *status*.** If this command needs a
directory the project's README does not list, ask before creating it, and add
the row to the README when the user agrees; when running autonomously, stop and
report instead. Never move a document because its status changed, and never
create a `completed/` directory.

- **Dated** (`YYYY-MM-DD-slug.md`) means *snapshot*: true as of that date, not
  edited afterwards except to correct an error.
- **Undated** (`slug.md`) means *living*: edited in place, deleted when it stops
  being true.
- **Reuse the slug** across feature → research → plan → verification. Same slug
  = same thread of work; the dates differ, the slug shouldn't.
- **Never move a document because its status changed**, and never create a
  `completed/` directory. A finished plan stays in `thoughts/plans/` — moving it
  rots every link that verifications and research wrote to it.

## Plan status: frontmatter, closed vocabulary

Every plan opens with YAML frontmatter. `status` is required and must be one of
four values; nothing else is a valid status:

```yaml
---
status: proposed | active | done | superseded
verified: thoughts/verifications/YYYY-MM-DD-slug.md   # when status: done
superseded_by: thoughts/plans/YYYY-MM-DD-slug.md      # when status: superseded
---
```

- **proposed** — written, not agreed. Nobody is working on it.
- **active** — being implemented right now.
- **done** — implemented. Link the verification.
- **superseded** — overtaken. Link the replacement; keep the file, because
  knowing what we decided *not* to do is worth as much as knowing what we did.

`grep -l "^status: active" thoughts/plans/*.md` has to answer "what is in
flight" truthfully — that is the whole point of putting status in frontmatter.
So a new plan is **`proposed`**, even when the user has agreed to it in
principle: it becomes `active` only when implementation actually starts.

Only plans carry `status:`. Research and verification documents don't — they are
dated snapshots and their date is their status.

Note the two levels: the frontmatter `status:` describes the **whole plan**, while
each phase carries its own `Implementation Status:` line (`Not Started` /
`In Progress` / `Complete`) in the body. They are different fields; don't collapse
them.

## Process

### 0. Project conventions

Before anything else, read the project's `CLAUDE.md` `## Workflow` section and,
if it exists, `.claude/workflow/create_plan.md`. They override the defaults in this
command: the checks to run (fast and full tiers, and the preconditions and
warnings listed with them, which are instructions), how the app starts, the CI
policy, tool pins, any rule under `### Thoughts` about writing there, and any
rule the project adds. If neither exists, proceed on this command's own
defaults, mention once that `bash ~/.claude/bin/init-project` sets the project
up, and do not invent project facts (check commands, hosts, credentials): ask
for them or leave them as open items. When running autonomously with no user
to ask, do not block:
derive what you can from the documents, mark every such decision "deduced,
not confirmed", and list it under the command's undecided or deferred section.

### 1. Load context

- If a research doc path was provided as argument, read it fully
- If not, list recent files in `thoughts/research/` and ask which to use
- If no research doc exists, suggest running `/research_codebase` first (and
  `/plan_feature` before that, if this is a new feature with no feature document)
- If the research document has a `**Feature:**` line, read that feature document
  fully too: its Desired Outcome and Success Measures are what the plan must
  deliver, and its Not In Scope list seeds "What We're NOT Doing"
- Read all files referenced in the research document

### 2. Ask for additional context

- Ask the user if there are additional instructions or context that should be taken into the plan
- Review the new information against the loaded context and identify any conflicts, new decisions needed, or open questions
- Do additional research in code base or search the Internet if needed

### 3. Resolve open decisions

Walk through each item in the research doc's "Decisions Needed" section:
- Present the options with their trade-offs
- Ask the user to choose
- Record the decision and rationale
- If a decision triggers new questions, research them before continuing

Do NOT proceed to planning until all decisions are resolved.

### 4. Design the implementation approach

Present a phased outline to the user BEFORE writing details:

```
## Proposed phases:
1. [Phase name] - [what it accomplishes]
2. [Phase name] - [what it accomplishes]
3. [Phase name] - [what it accomplishes]

Does this phasing make sense?
```

Get feedback and adjust before writing the draft.

### 5. Write the draft plan

Save to `thoughts/plans/YYYY-MM-DD-slug.md` with `status: proposed`, reusing the
research document's slug and using the format in step 7. Leave "Evaluation
Notes", "Decisions Deferred", "What Already Exists", and each phase's "Failure
Scenarios" empty for now; step 6 fills them. The draft exists as a file so that
the evaluators review the actual text, not a summary.

### 6. Evaluate the draft

Three passes, in this order, all against the draft plan file and the actual
codebase.

**Pass 1: checks you run yourself, before spawning anyone**

- **Minimum change set.** For each sub-problem, what existing code already
  solves it partly or fully, and does the plan reuse it or rebuild it? Flag any
  work that could be deferred without blocking the outcome. If the plan touches
  many files or introduces several new services or classes, challenge whether
  the same outcome needs fewer moving parts.
- **Failure scenarios.** For each new codepath or integration point, name one
  realistic production failure (timeout, missing value, race, stale data) and
  check whether the plan has a test for it, error handling for it, and whether
  the user would see a clear error or a silent failure. No test, no handling,
  and silent is a critical gap: add it to the plan.
- **Rationale.** Every item under "What We're NOT Doing" gets a one-line reason.
  A deferral without a reason is a guess.

**Pass 2: evaluator team**

Spawn five agents in parallel. Each reviews the draft against the actual
codebase and returns specific concerns:

- **Architecture**: Does this fit existing patterns in the codebase? Integration risks? Dependencies? Single points of failure?
- **Security**: Authentication, authorization, injection, or data exposure risks?
- **Scalability**: Performance under load? Database query patterns? Memory usage?
- **Usability**: Is the API/UI intuitive? Error messages helpful? Migration path clear?
- **Tests and code quality**: Trace every new codepath (where input comes from,
  what transforms it, where it goes, what can go wrong at each step) and check
  each branch against the tests that exist and the tests the plan adds. If the
  plan fixes a defect, a regression test is a required item, not a suggestion.
  Flag duplication of existing code, missing error and edge handling, and
  anything over- or under-engineered for the outcome.

**Evidence rule, for every evaluator:** a concern must quote the line that
motivates it, as `file:line` plus the text. A concern that cannot quote a line
is a note, not a finding, and goes in Evaluation Notes as "unverified".

**Pass 3: outside voice**

After the five return, use the **second-opinion** capability
(`~/.claude/tools/second-opinion.md`) through the **outside-voice** agent
(`~/.claude/agents/outside-voice.md`). Give it only the plan path and the
evaluators' findings. Its brief is fixed: find what the review missed, logical
gaps and unstated assumptions, a fundamentally simpler approach, feasibility
risks taken for granted, missing dependencies or sequencing problems, and
whether this is the right thing to build at all. It runs a second model when
one is installed and is the fresh-context reviewer otherwise, and says which;
record that under "Outside voice" in Evaluation Notes.

**Incorporate the findings**

- A finding that changes scope, phasing, or a recorded decision goes to the
  user as a choice with options and a recommendation. Get the decision.
- A mechanical finding (a missing test, a reuse opportunity, a rename) is folded
  into the plan directly and listed in Evaluation Notes.
- If the user explicitly defers a decision, record it under "Decisions
  Deferred" with what it blocks. Never silently pick a default.
- Once the user has accepted or rejected a scope change, commit to it. Do not
  re-argue it in a later pass.

### 7. Finalize the plan

Update `thoughts/plans/YYYY-MM-DD-slug.md` to this format:

```markdown
---
status: proposed
---

# Implementation Plan: [Feature Name]

**Date:** YYYY-MM-DD
**Feature:** thoughts/features/YYYY-MM-DD-slug.md   (when there is one; omit otherwise)
**Research:** thoughts/research/YYYY-MM-DD-slug.md
**Branch:** [current branch]

## Overview

[What we're building and why, 2-3 sentences. When there is a feature document,
name the outcome and the headline success measure it commits to.]

## Decisions Made

- **[Decision 1]**: [Choice] - [rationale]
- **[Decision 2]**: [Choice] - [rationale]

## Decisions Deferred

(Omit when empty.)

- **[Decision]**: deferred by [user] on YYYY-MM-DD - [what it blocks, and when it must be made]

## What We're NOT Doing

- [Explicit out-of-scope item] - [one-line reason]
- [Another out-of-scope item] - [one-line reason]

## What Already Exists

- `path/to/existing.py` - [what it already solves, and whether the plan reuses it]

## Phase 1: [Descriptive Name]

### Changes

- `path/to/file.py` - [what changes and why]
- `path/to/other.py` - [what changes and why]

### New Tests, both unit and integration tests

- [Test description and what it verifies]
- [Another test]

### Failure Scenarios

- [New codepath] - [realistic failure] - covered by [test / handling / neither: added above]

### Verification

- The commands from the contract's `### Checks`; these are examples for a python code base:
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

### Tests and code quality
[Key feedback and how it was addressed]

### Outside voice
[What the fresh-context review found, and what was done about it]

### Unverified notes
[Concerns that could not cite a line, kept for the implementer's awareness]
```

**Diagrams.** A phase with a non-trivial data flow, state machine, or
pipeline gets a diagram in the plan, using the **diagram** capability
(`~/.claude/tools/diagram.md`). A mermaid fence is always enough.

### 8. Present for approval

Show the user the plan location and a summary. Ask if they want to adjust anything before implementation.

## Guidelines

- **Always write the `status:` frontmatter block** - a plan without it is
  invisible to the greps the convention exists to serve
- **Don't restate the frontmatter in a prose status line** - add one only when
  there is detail the four values can't carry ("Phases 1-3 done, 4 blocked on an
  upstream release"), and keep it consistent with the frontmatter, which always wins
- **Do not switch into plan mode** - we want all plans and documents in the repository
- **Do not make any code changes** - this step is just to plan out what to do
- **Each phase must be independently testable** - don't create phases that leave things broken
- **Include specific file paths** for all planned changes
- **Every phase needs new tests** for new functionality AND passing existing tests
- **No open questions in the final plan** - resolve everything with the user
  first. A decision the user explicitly defers is recorded under "Decisions
  Deferred" with what it blocks; it is never left as a question or silently defaulted
- **Findings need evidence** - an evaluator concern that cannot quote the
  motivating line is a note, not a finding
- **Separate automated verification** (commands to run) from manual verification
- **Keep phases small** - prefer more small phases over fewer large ones
