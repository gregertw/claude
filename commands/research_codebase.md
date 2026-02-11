# Research Codebase

Investigate the codebase deeply and research the web to produce a factual document useful for decision-making.

## Process

### 1. Understand the question

- If a file path or topic was provided as argument, read it fully and begin research immediately
- If no argument provided, ask: "What would you like me to research? Provide a question, topic, or path to a file with context."

### 2. Read all mentioned files and get context

- Read every referenced file FULLY (no limit/offset) before spawning sub-agents
- This ensures complete context before decomposing the research
- Read CLAUDE.md and identify any relevant code structure, instructions, or repo specifics that are relevant

### 3. Decompose and research in parallel

Break the question into independent research areas. Spawn sub-agents concurrently:

- **Explore** agents to find relevant files, patterns, and directory structure
- **codebase-analyzer** agents to understand specific implementations in detail
- **web-search-researcher** agents for external documentation, best practices, and alternatives

Each agent should return specific `file:line` references and concrete findings.

Wait for ALL sub-agents to complete before proceeding.

### 4. Synthesize findings

- Cross-reference sub-agent findings, resolve contradictions
- Identify decisions that need to be made
- For each decision, lay out concrete options with pros and cons
- Do NOT recommend improvements or critique the code - be factual about what exists and what the options are

### 5. Write the research document

Save to `thoughts/research/YYYY-MM-DD-description.md`:

```markdown
# Research: [Topic]

**Date:** YYYY-MM-DD
**Status:** Complete
**Branch:** [current branch]
**Commit:** [current short hash]

## Research Question

[Original question]

## Summary

[2-3 paragraph summary of key findings]

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

### 6. Present findings

Show the user a concise summary with the document path and highlight the key decisions that need answers.

## Guidelines

- **Be factual** - describe what IS, not what SHOULD be
- **Include specific file paths and line numbers** for all claims
- **Always include web research** for external context, best practices, and alternatives
- **Lay out decisions clearly** - this document feeds into `/create_plan`
- **Use sub-agents liberally** - parallel research is faster and saves context
- **Read all mentioned files FULLY** before spawning sub-agents
