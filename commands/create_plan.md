# Create Implementation Plan

Take a research document, interact with the user to resolve decisions, and produce an actionable implementation plan. Evaluate the plan from multiple perspectives before finalizing.

## Documents and plan state

Layout, slugs, plan frontmatter (`status`, `approved`, `verified`,
`iterations`), acceptance criteria, traceability and living specs are defined in
`~/.claude/shared/conventions.md`. What this command owns:

- A new plan is **`proposed`**, even when the user has agreed in principle; it
  becomes `active` only when `/implement_plan` starts it.
- **`approved:`** is written only when the user explicitly approves the
  finished plan (step 8). Never infer it, and never write it when running
  autonomously.
- The plan reuses the research document's slug; in the small-change lane it
  mints one that names the outcome.

## Process

### 0. Project conventions

Read `~/.claude/shared/conventions.md` and follow its Project conventions
section; this command's addendum is `.claude/workflow/create_plan.md`. The rest of
that file defines the document layout, plan frontmatter, acceptance
criteria, living specs and the iterations log this command refers to.

### 1. Load context

- If a research doc path was provided as argument, read it fully
- If not, list recent files in `thoughts/research/` and ask which to use
- If no research doc exists, suggest running `/research_codebase` first (and
  `/plan_feature` before that, if this is a new feature with no feature
  document), unless the change qualifies for the **small-change lane** below
- If the research document has a `**Feature:**` line, read that feature document
  fully too: its Desired Outcome and Success Measures are what the plan must
  deliver, its User Experience and Acceptance Criteria are what the phases must
  cover, and its Not In Scope list seeds "What We're NOT Doing"
- Read all files referenced in the research document
- Read the living spec for every capability the plan touches
  (`thoughts/specs/`, or the location the contract names). The plan's Spec
  Delta is written against it. A `draft` spec's statements are claims to
  check against the code, not facts. A capability with no spec: offer
  `/create_spec <capability>` first, so the delta has a baseline; if the user
  declines, the delta's statements will start a `partial` spec at merge

**Small-change lane.** When the argument is a description rather than a
research document, and the change is one phase, about five files or fewer, and
adds no data model, migration, auth or permission rule, or external
integration, propose the small-change lane and let the user confirm it. In the
lane: no research document; the plan writes its own acceptance criteria (step
4); step 6 runs Pass 1 and Pass 3 but not the evaluator team; and the plan says
`**Lane:** small change` under its header. Everything else applies: decisions,
acceptance criteria, traceability, spec delta, approval. If the work grows past
the lane while planning, say so and stop for research.

### 2. Ask for additional context

- Ask the user if there are additional instructions or context that should be taken into the plan
- Review the new information against the loaded context and identify any conflicts, new decisions needed, or open questions
- Do additional research in code base or search the Internet if needed

### 3. Resolve open decisions

Walk through each item in the research doc's "Decisions Needed" section (in
the small-change lane: the decisions the description leaves open):
- Present the options with their trade-offs
- Ask the user to choose
- Record the decision and rationale
- If a decision triggers new questions, research them before continuing

Move on to planning once every decision is resolved or explicitly deferred by the user (recorded under Decisions Deferred).

### 4. Design the implementation approach

Settle the acceptance criteria first. Start from the feature document's list
(or write one with the user when there is none, in the format of
`~/.claude/shared/conventions.md`). Decisions made in step 3 may add, change or
drop criteria: added ones continue the numbering and say `(added in plan:
reason)`; dropped ones stay, struck through, with the reason. Every UX path in
the feature document must be covered by at least one AC.

Then present a phased outline to the user BEFORE writing details, with the
criteria each phase delivers:

```
## Proposed phases:
1. [Phase name] - [what it accomplishes] - AC-1, AC-2
2. [Phase name] - [what it accomplishes] - AC-3
3. [Phase name] - [enabling work, no AC: why it is needed]

Does this phasing make sense?
```

Get feedback and adjust before writing the draft.

### 5. Write the draft plan

Save to `thoughts/plans/YYYY-MM-DD-slug.md` with `status: proposed`, reusing the
research document's slug (or the minted one in the small-change lane) and using the format in step 7. Leave "Evaluation
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
- **Coverage.** Every AC appears in the Traceability table with a phase and a
  named test (`path::test_name`), or a named manual check with the reason no
  automated test is possible. Every UX path of the feature document maps to at
  least one AC. An AC without a test is a critical gap: add the test to the
  phase.
- **Spec delta.** Every behaviour the plan adds, changes or removes appears in
  the Spec Delta against the current spec's wording, and every decision that
  users or later plans must respect is listed there too.
- **Size.** The plan describes behaviour, files and interfaces, not code. Code
  blocks only for schemas, signatures and wire formats. Over roughly 600 lines
  before Evaluation Notes, look for a phase that is really its own plan, or
  detail that belongs in research.

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
  For each AC, would the named test actually fail if the behaviour were
  missing? A test that only exercises the code path is not coverage.
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
# approved: YYYY-MM-DD by <who>   (added in step 8, only on explicit approval)
---

# Implementation Plan: [Feature Name]

**Date:** YYYY-MM-DD
**Feature:** thoughts/features/YYYY-MM-DD-slug.md   (when there is one; omit otherwise)
**Research:** thoughts/research/YYYY-MM-DD-slug.md   (omit in the small-change lane)
**Lane:** small change   (only in that lane)
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

## Acceptance Criteria

- **AC-1** (main path, step 1) — When [trigger], [observable result].
- **AC-2** (edge: [situation]) — When [trigger], [observable result].
- **AC-5** (added in plan: [decision that introduced it]) — ...
- ~~**AC-4**~~ — dropped: [reason]

## Traceability

| AC | Phase | Test | Status |
| --- | --- | --- | --- |
| AC-1 | 1 | `tests/path/test_x.py::test_name` | Not run |
| AC-2 | 2 | manual: [browser walk, step] — [why no automated test] | Not run |

## Spec Delta

### `thoughts/specs/[capability].md`   (new spec: say so)

- **ADDED** ([Area]) — When [trigger], [observable result]. (AC-1)   (ID assigned at merge)
- **MODIFIED SHR-4** — "[current wording]" → "[new wording]". (AC-2)
- **REMOVED SHR-7** — "[current wording]" — [why]
- **DECISION** — [decision users or later plans must respect, one line]

## Phase 1: [Descriptive Name]

### Changes

- `path/to/file.py` - [what changes and why]
- `path/to/other.py` - [what changes and why]

### New Tests (written first: `/implement_plan` sees them fail before coding)

- `tests/path/test_x.py::test_name` — AC-1: [what it asserts]
- `tests/path/test_y.py::test_other` — failure scenario: [what it guards]

### Failure Scenarios

- [New codepath] - [realistic failure] - covered by [test / handling / neither: added above]

### Verification

- [ ] Run the contract's `### Checks` fast tier verbatim
- [ ] [manual verification step if needed]

### Implementation Status: Not Started

---

## Phase 2: [Descriptive Name]

[Same structure...]

---

## Evaluation Notes

**Outside voice:** [provider, or fresh-context reviewer]

| # | From | Finding (`file:line` quoted) | Disposition |
| --- | --- | --- | --- |
| 1 | Security | `api/share.py:42` "…" — [concern] | Folded into Phase 2 / User decided: [choice] / Rejected: [why] |

**Unverified notes** (concerns that could not cite a line):
- [note]
```

**Diagrams.** A phase with a non-trivial data flow, state machine, or
pipeline gets a diagram in the plan, using the **diagram** capability
(`~/.claude/tools/diagram.md`). A mermaid fence is always enough.

### 8. Present for approval

Show the user the plan location and a summary: phases, the number of
acceptance criteria and any covered only by a manual check, the specs the
delta touches, and anything deferred. Ask whether to adjust anything, or
whether they approve it for implementation.

On an explicit approval, add `approved: YYYY-MM-DD by <who>` to the
frontmatter and leave `status: proposed`. "Looks good" to a question about
one section is not approval of the plan; ask. When running autonomously, never
write `approved:`; report that the plan awaits approval.

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
- **Every AC has a test before the plan is approved** - traceability is decided
  here, not discovered at verification
- **Behaviour, not code** - a reviewer should be able to approve the plan
  without reading code blocks; keep them to schemas, signatures and wire formats
- **The plan never edits a spec** - it carries a Spec Delta, which
  `/verify_implementation` merges when the plan lands
