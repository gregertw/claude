# Update Implementation Plan

Take new context, facts, or data and update an existing implementation plan that was created with `/create_plan`.

## Arguments

$ARGUMENTS - Optional: path to the plan file to update, or new context/instructions

## Plan status while updating

The plan's YAML frontmatter carries its status, from a closed four-value
vocabulary (`proposed` | `active` | `done` | `superseded`). This command works on
plans that haven't been implemented, so:

- **Normally the status doesn't change** — a `proposed` plan being revised is
  still `proposed`. Record what changed in the Update Log, not in the status.
- **If the update is so large it replaces the plan rather than revising it**,
  write a new plan and set the old one to `status: superseded` with
  `superseded_by: thoughts/plans/YYYY-MM-DD-slug.md`. Keep the superseded file:
  knowing what we decided *not* to do is worth as much as knowing what we did.
- **Never move the plan file** and never create a `completed/` directory — a
  directory is a *kind* of document, never a *status*. Moving a plan rots the
  links that research and verifications wrote to it.

Only plans carry `status:`. Research and verification documents don't.

## Process

### 0. Project conventions

Before anything else, read the project's `CLAUDE.md` `## Workflow` section and,
if it exists, `.claude/workflow/update_plan.md`. They override the defaults in this
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

For phases that need significant changes, run the same evaluation as
`/create_plan` step 6, scoped to the changed phases:

- First, yourself: re-check the minimum change set (does the change rebuild
  something that exists?), add a Failure Scenario for each new codepath, and
  give every new "What We're NOT Doing" item a one-line reason.
- Then spawn the five evaluators in parallel:
  - **Architecture**: Do changes fit existing patterns? New integration risks?
  - **Security**: New authentication, authorization, or data exposure concerns?
  - **Scalability**: Performance implications of changes?
  - **Usability**: API/UI impact? Migration path still clear?
  - **Tests and code quality**: Are the new codepaths traced and tested? Regression test if a defect is being fixed? Duplication, missing error and edge handling?
- Evidence rule: a concern must quote the motivating line as `file:line`; one that cannot is a note under "Unverified notes".
- Finally, the **outside-voice** agent (`~/.claude/agents/outside-voice.md`, the second-opinion capability) given only the plan path and the evaluators' findings: what did the review miss?

Findings that change scope, phasing, or a recorded decision go to the user;
mechanical findings fold in. A decision the user explicitly defers is recorded
under "Decisions Deferred", never silently defaulted.

Only evaluate phases with significant changes - skip this for minor updates.

### 6. Update the plan

Modify the existing plan file in place. Preserve the original structure and:

- Leave the frontmatter `status:` alone unless the plan is being replaced
  wholesale (then: `superseded` + `superseded_by:`, per above). If the plan
  predates the convention and has no frontmatter, add it — `status: proposed` for
  a plan that hasn't been started
- Add an **Update Log** section at the top (after the frontmatter and header
  metadata) if one doesn't exist:

```markdown
## Update Log

- **YYYY-MM-DD**: [Brief description of what changed and why]
```

- If an Update Log already exists, append the new entry

- Update **Decisions Made** section:
  - Keep existing decisions that are still valid
  - Mark changed decisions with "[Updated YYYY-MM-DD]"
  - Add new decisions

- Update **What We're NOT Doing** if scope changed, with a one-line reason per item

- Update **Decisions Deferred** and **What Already Exists** if the change touched them

- Update affected phases:
  - Modify existing phase content as needed
  - Add new phases if required
  - If a phase needs to be removed, move it to a "Removed Phases" section with rationale rather than deleting it
  - Update **Implementation Status** on each phase to reflect current state

- Update **Evaluation Notes** with any new evaluation feedback, including the
  "Tests and code quality", "Outside voice", and "Unverified notes" subsections

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
