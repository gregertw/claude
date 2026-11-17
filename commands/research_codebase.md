# Research Codebase

Investigate the codebase deeply and research the web to produce a factual document useful for decision-making.

Two ways in:

- **From a feature document** (`/plan_feature` output in `thoughts/features/`):
  the research tests the feature's hypothesis against the codebase and answers
  its open questions. The desired outcome stays the anchor; the code is what we
  measure it against. This is the normal path for a new feature.
- **From a question or topic**: the research describes what exists and lays out
  the options. Use this for investigations, bugs, and technical questions that
  are not a new feature.

## Arguments

$ARGUMENTS - Optional: path to a feature document, path to a file with context, or a question/topic

## Where documents live

Layout and the dated/living rules: `~/.claude/shared/conventions.md`
("Where documents live").

Research goes in `thoughts/research/YYYY-MM-DD-slug.md`. The **date is the
status**: a research document is a snapshot, true as of the day it was written,
and it is not edited afterwards except to correct an error. If you find yourself
wanting to keep a document current, it belongs in `thoughts/reference/` (living,
undated) instead — that includes durable findings that outlive the question that
prompted them. A protocol flow or state machine that ends up in reference gets a
diagram through the **diagram** capability (`~/.claude/tools/diagram.md`); a
mermaid fence is always enough.

**Reuse the feature document's slug when there is one; mint it otherwise.** The
plan and the verification will reuse it too. Same slug across the directories is
how a piece of work is followed end to end.

Research documents carry **no `status:` frontmatter** — only plans do. Don't
invent one.

## Process

### 0. Project conventions

Read `~/.claude/shared/conventions.md` and follow its Project conventions
section; this command's addendum is `.claude/workflow/research_codebase.md`. The rest of
that file defines the document layout, plan frontmatter, acceptance
criteria, living specs and the iterations log this command refers to.

### 1. Understand the question

- If the argument is a path under `thoughts/features/`, read it fully. The
  research is now feature-driven: reuse the feature's slug, and treat its
  Desired Outcome, User Experience, Acceptance Criteria, Success Measures,
  Hypothesis, and Open Questions as the source of the research questions (step 3)
- If any other file path or a topic was provided, read it fully and begin research immediately
- If no argument provided, ask: "What would you like me to research? Provide a
  feature document, a question, a topic, or a path to a file with context." If
  the answer describes a new feature rather than a question, suggest running
  `/plan_feature` first so the outcome is written down before the code is looked at

### 2. Read all mentioned files and get context

- Read every referenced file FULLY (no limit/offset) before spawning sub-agents
- This ensures complete context before decomposing the research
- Read CLAUDE.md and identify any relevant code structure, instructions, or repo specifics that are relevant
- Read the living spec for each capability the topic touches (`thoughts/specs/`, or the location the contract names), if one exists

### 3. Formulate the research questions

Write down the concrete questions the research must answer before spawning
anything. Each question should be answerable with evidence, not a theme.

When feature-driven, derive them from the feature document. Cover at least:

- **Open Questions** — every question listed there, verbatim or sharpened
- **Hypothesis** — for each moving part in the hypothesis: does something like it
  exist already, what would it touch, and what in the current code contradicts it?
- **User Experience** — for each step of the main and edge paths: which existing
  screens, endpoints, or flows does it pass through, and what constrains it?
- **Acceptance Criteria** — for each AC: which code path would deliver it,
  and at what level can it be tested (unit, integration, browser), with which
  existing fixtures, factories, or harness? An AC that no level can test is a
  finding: the plan must either add the harness or name a manual check.
- **Current spec** — for each capability touched: what does its living spec
  say today, which statements would the hypothesis change, and where does the
  code already disagree with the spec? (No spec yet: say so; the plan's Spec
  Delta will start one.)
- **Success Measures** — for each measure: can it be observed with the
  instrumentation, logging, or data that exists today? If not, what is missing?
- **Documentation** — for each moving part: which pages of the project's
  user-facing docs (the docs tree, README, example READMEs, guides, API and
  protocol references, troubleshooting) state the behaviour it changes, and
  which statements become false, which need a new section, and which are
  already wrong against the code? Sweep the whole docs tree per item (grep
  the names, config keys, routes, error codes and defaults it touches), not
  only the pages the code happens to point at. Released-version migration
  guides and changelogs are historical; list them only to say they stay.

When question-driven, break the question into independent areas the same
way, including the documentation question for anything that changes
behaviour.

For each question, decide how it gets answered:

- **From context already read** — answer it directly, no agent
- **Needs the codebase** — an **Explore** agent (find where things are) or a
  **codebase-analyzer** agent (understand how something works)
- **Needs the outside world** — a **web-search-researcher** agent

Not every question needs an agent, and closely related questions should share
one. Show the user the list of questions and who answers each before spawning,
so a missing or misread question is caught cheaply:

```
## Research questions
1. [Question] — from context
2. [Question] — codebase-analyzer
3. [Question] — Explore
4. [Question] — web-search-researcher

Anything missing or off target?
```

### 4. Research in parallel

Spawn one sub-agent per question (or per group of closely related questions) concurrently:

- **Explore** agents to find relevant files, patterns, and directory structure
- **codebase-analyzer** agents to understand specific implementations in detail
- **web-search-researcher** agents for external documentation, best practices, and alternatives (the web-search capability, `~/.claude/tools/web-search.md`; honor a `Web search: none` pin in `CLAUDE.md`)

Each agent should return specific `file:line` references and concrete findings.

Give each agent the question it owns, the relevant excerpt of the feature
document (outcome and hypothesis, so it knows what "relevant" means), and the
files already read. Wait for ALL sub-agents to complete before proceeding.

### 5. Synthesize findings

- Cross-reference sub-agent findings, resolve contradictions
- Answer each research question explicitly; if one could not be answered, say what is missing
- When feature-driven, state plainly where the code supports the hypothesis and
  where it contradicts it — as facts about the code, not as a verdict on the feature
- Identify decisions that need to be made
- For each decision, lay out concrete options with pros and cons
- Do NOT recommend improvements or critique the code - be factual about what exists and what the options are

### 6. Write the research document

Save to `thoughts/research/YYYY-MM-DD-slug.md`:

```markdown
# Research: [Topic]

**Date:** YYYY-MM-DD
**Feature:** thoughts/features/YYYY-MM-DD-slug.md   (when feature-driven; omit otherwise)
**Branch:** [current branch]
**Commit:** [current short hash]

## Research Questions

1. **[Question]** — [one-line answer, with a pointer to the section below]
2. **[Question]** — [one-line answer]
3. **[Question]** — [unanswered: what is missing]

## Summary

[2-3 paragraph summary of key findings. When feature-driven: how the codebase
relates to the hypothesis, what the user experience would pass through, and
which success measures are observable today.]

## Detailed Findings

### [Area 1]

[Findings with `file:line` references]

### [Area 2]

...

## Acceptance Criteria Testability

(When feature-driven; omit otherwise.)

| AC | Code path | Test level and harness |
| --- | --- | --- |
| AC-1 | `path/to/handler.py:40` → `path/to/store.py:88` | integration, existing `client` fixture |
| AC-3 | ... | none today: [what is missing] |

## Documentation Impact

### [Moving part 1]

- `docs/path/page.rst:12-30` - [what it states today; false after the change /
  needs a new section / already wrong against the code / stays]

[One subsection per moving part. Say "none" explicitly when a sweep found no
page, so the absence is a finding rather than an omission.]

## Decisions Needed

### Decision 1: [Question]

**Options:**
1. **[Option A]** - [pros/cons with concrete evidence]
2. **[Option B]** - [pros/cons with concrete evidence]

**Recommendation:** [only if evidence clearly favors one option]

### Decision 2: [Question]

...

## Code References

- `path/to/file.py:123` - Description
- `path/to/other.py:45-67` - Description

## External References

- [URL] - Description of what was found
```

### 7. Present findings

Show the user a concise summary with the document path and highlight the key
decisions that need answers. When feature-driven, also say which open questions
from the feature document are still unanswered. Then point to the next step:

```
Next: /create_plan thoughts/research/YYYY-MM-DD-slug.md
```

## Guidelines

- **Identify decisions, don't make them** — lay out options with evidence; add a Recommendation only where the evidence clearly favours one, and leave the choice to `/create_plan`
- **Be factual** - describe what IS, not what SHOULD be
- **Include specific file paths and line numbers** for all claims
- **Sweep the docs, not just the code** - every behaviour change gets a
  documentation question in step 3 and a `## Documentation Impact` entry; a
  page the code never points at (a guide's example output, a troubleshooting
  entry, a protocol spec sentence) is exactly the one a code-led search misses
- **Include web research where the outside world bears on a question** (external docs, known issues, alternatives), unless `### Tools` pins `Web search: none`; then record those questions as open
- **Lay out decisions clearly** - this document feeds into `/create_plan`
- **When feature-driven, keep the outcome as the anchor** - the question is
  "what does the code mean for this outcome and hypothesis", not "what does the
  code do". Do not let the research drift into describing the codebase for its own sake
- **Do not edit the feature document or the spec** - the feature document is a dated snapshot, and the spec changes only through the commands that own it (a plan's Spec Delta, merged at verification, and after that iterations and bug fixes). A hypothesis the
  code contradicts becomes a finding here and a decision in the plan, not a rewrite there
- **Write the questions before spawning agents** - an agent with a precise
  question returns evidence; one with a topic returns a tour
- **Use sub-agents for every question that needs one** - parallel research is faster and saves context; a question answerable from what is already read does not need one
- **Read all mentioned files FULLY** before spawning sub-agents
