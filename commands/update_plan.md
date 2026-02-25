# Update Implementation Plan

Take new context, facts, or data and update an existing implementation plan that was created with `/create_plan`.

## Arguments

$ARGUMENTS - Optional: path to the plan file to update, or new context/instructions

## Process

### 1. Identify the plan to update

- If a plan path was provided as argument, read it fully
- If not, list recent files in `thoughts/plans/` and ask which to update
- Read the full plan file and its referenced research document (if any)
- This is to change a plan before it's implemented. If the plan shows implementation has started, stop and tell the user that /iterate_plan should be used to work on the implementation.

### 2. Gather new context

- Ask the user what has changed or what new information should be incorporated
- If the argument contains new context (not just a path), use that as the starting point
- Understand whether this is:
  - **New requirements** - additional features or constraints
  - **Changed decisions** - reversing or modifying earlier choices
  - **New facts** - discovered during implementation or research
  - **Scope changes** - adding or removing phases/features
  - **Implementation feedback** - things that didn't work as planned

### 3. Assess impact

For each piece of new information, determine:
- Which phases are affected?
- Do any completed phases need rework?
- Are new phases needed?
- Do any decisions need to be revisited?
- Are there new conflicts or open questions?

Present a summary of the impact to the user:

```
## Impact Assessment

### Affected Phases
- Phase N: [what changes and why]
- Phase M: [what changes and why]

### New Decisions Needed
- [Decision question]

### Phases Unaffected
- Phase X, Phase Y (no changes needed)

Does this assessment look right?
```

Get confirmation before making changes.

### 4. Resolve new decisions

If the update introduces new open questions or invalidates previous decisions:
- Present each decision with options and trade-offs
- Ask the user to choose
- Record the decision and rationale

Do NOT proceed to updating the plan until all new decisions are resolved.

### 5. Re-evaluate affected phases

For phases that need significant changes, spawn agents to evaluate from the same four perspectives as `/create_plan`:

- **Architecture**: Do changes fit existing patterns? New integration risks?
- **Security**: New authentication, authorization, or data exposure concerns?
- **Scalability**: Performance implications of changes?
- **Usability**: API/UI impact? Migration path still clear?

Only evaluate phases with significant changes - skip this for minor updates.

### 6. Update the plan

Modify the existing plan file in place. Preserve the original structure and:

- Update the **Status** field if appropriate (e.g., "Updated YYYY-MM-DD" or keep "Ready for Implementation")
- Add an **Update Log** section at the top (after the header metadata) if one doesn't exist:

```markdown
## Update Log

- **YYYY-MM-DD**: [Brief description of what changed and why]
```

- If an Update Log already exists, append the new entry

- Update **Decisions Made** section:
  - Keep existing decisions that are still valid
  - Mark changed decisions with "[Updated YYYY-MM-DD]"
  - Add new decisions

- Update **What We're NOT Doing** if scope changed

- Update affected phases:
  - Modify existing phase content as needed
  - Add new phases if required
  - If a phase needs to be removed, move it to a "Removed Phases" section with rationale rather than deleting it
  - Update **Implementation Status** on each phase to reflect current state

- Update **Evaluation Notes** with any new evaluation feedback

### 7. Present changes

Show the user a concise summary of what was changed:

```
## Plan Updated: [plan file path]

### Changes Made
- [Change 1]
- [Change 2]

### New/Modified Decisions
- [Decision]: [New choice] - [rationale]

### Phases Modified
- Phase N: [summary of changes]

### Next Steps
- [What should happen next]
```

Ask if they want to adjust anything.

## Guidelines

- **Preserve completed work** - never remove or invalidate phases marked as completed without explicit user approval
- **Maintain traceability** - the Update Log should make it clear what changed and when
- **Keep phases independently testable** - don't break this property when updating
- **Minimize disruption** - prefer modifying existing phases over reorganizing the entire plan
- **Include specific file paths** for all new or changed planned modifications
- **Every new or modified phase needs tests** for new functionality AND passing existing tests
- **No open questions in the updated plan** - resolve everything with the user first
