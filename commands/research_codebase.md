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

The layout of `thoughts/` is defined by the project's `thoughts/README.md`,
which is authoritative; the default it starts from is
`~/.claude/templates/thoughts-README.md`: `features/`, `research/`, `plans/`,
`verifications/` (dated snapshots) and `reference/`, `todo/` (living). **A
directory is a *kind* of document, never a *status*.** If this command needs a
directory the project's README does not list, ask before creating it, and add
the row to the README when the user agrees; when running autonomously, stop and
report instead. Never move a document because its status changed, and never
create a `completed/` directory.

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

Before anything else, read the project's `CLAUDE.md` `## Workflow` section and,
if it exists, `.claude/workflow/research_codebase.md`. They override the defaults in this
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

### 1. Understand the question

- If the argument is a path under `thoughts/features/`, read it fully. The
  research is now feature-driven: reuse the feature's slug, and treat its
  Desired Outcome, User Experience, Success Measures, Hypothesis, and Open
  Questions as the source of the research questions (step 3)
- If any other file path or a topic was provided, read it fully and begin research immediately
- If no argument provided, ask: "What would you like me to research? Provide a
  feature document, a question, a topic, or a path to a file with context." If
  the answer describes a new feature rather than a question, suggest running
  `/plan_feature` first so the outcome is written down before the code is looked at

### 2. Read all mentioned files and get context

- Read every referenced file FULLY (no limit/offset) before spawning sub-agents
- This ensures complete context before decomposing the research
- Read CLAUDE.md and identify any relevant code structure, instructions, or repo specifics that are relevant

### 3. Formulate the research questions

Write down the concrete questions the research must answer before spawning
anything. Each question should be answerable with evidence, not a theme.

When feature-driven, derive them from the feature document. Cover at least:

- **Open Questions** — every question listed there, verbatim or sharpened
- **Hypothesis** — for each moving part in the hypothesis: does something like it
  exist already, what would it touch, and what in the current code contradicts it?
- **User Experience** — for each step of the main and edge paths: which existing
  screens, endpoints, or flows does it pass through, and what constrains it?
- **Success Measures** — for each measure: can it be observed with the
  instrumentation, logging, or data that exists today? If not, what is missing?

When question-driven, break the question into independent areas the same way.

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

- **Focus on research and identifying key decisions** - do not make recommendations or ask the user to make decisions
- **Be factual** - describe what IS, not what SHOULD be
- **Include specific file paths and line numbers** for all claims
- **Always include web research** for external context, best practices, and alternatives
- **Lay out decisions clearly** - this document feeds into `/create_plan`
- **When feature-driven, keep the outcome as the anchor** - the question is
  "what does the code mean for this outcome and hypothesis", not "what does the
  code do". Do not let the research drift into describing the codebase for its own sake
- **Do not edit the feature document** - it is a dated snapshot. A hypothesis the
  code contradicts becomes a finding here and a decision in the plan, not a rewrite there
- **Write the questions before spawning agents** - an agent with a precise
  question returns evidence; one with a topic returns a tour
- **Use sub-agents for every question that needs one** - parallel research is faster and saves context; a question answerable from what is already read does not need one
- **Read all mentioned files FULLY** before spawning sub-agents
