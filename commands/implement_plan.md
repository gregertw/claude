# Implement Plan

Execute an implementation plan phase by phase, ensuring tests pass at each step and keeping the plan document updated with progress and learnings.

## Plan status and where things go

The plan's YAML frontmatter carries its status, from a closed four-value
vocabulary (`proposed` | `active` | `done` | `superseded`). This command owns two
transitions:

- **On starting work:** set `status: active` in the frontmatter. `active` means
  *being implemented right now* — it is what
  `grep -l "^status: active" thoughts/plans/*.md` reports as in flight.
- **When all phases are complete:** set `status: done`. After
  `/verify_implementation` runs, the plan also gets
  `verified: thoughts/verifications/YYYY-MM-DD-slug.md`.

Two rules that matter more than they look:

- **Never leave a plan `active` once you stop working on it.** A plan whose
  remaining phases were abandoned is `done`, not `active` — write the unfinished
  remainder to `thoughts/todo/<slug>.md` (undated, living, deleted when the work
  lands) and let the plan close. Stalled `active` plans are what make the
  in-flight query worthless.
- **Never move the plan file**, and never create a `completed/` directory. A
  directory is a *kind* of document, never a *status*. The finished plan stays in
  `thoughts/plans/` because the verification links to it by path.

Things discovered while implementing:

- Real but out of scope now → `thoughts/todo/<slug>.md`, following any rule
  the contract's `### Thoughts` states for that directory (an index to update,
  a naming scheme)
- Durable knowledge about how the system works → `thoughts/reference/<slug>.md`,
  updated in place from then on
- Neither is dated — both are living documents.

## Process

### 0. Project conventions

Before anything else, read the project's `CLAUDE.md` `## Workflow` section and,
if it exists, `.claude/workflow/implement_plan.md`. They override the defaults in this
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

### 1. Load the plan

- If a plan path was provided as argument, read it fully
- If not, list recent files in `thoughts/plans/` and ask which to implement
- Read the plan completely, noting any phases already marked as done
- Read the referenced research document and all files mentioned in the plan

### 2. Resume from current state

- Look for phases marked with `Implementation Status: Complete` - skip those
- Start from the first phase marked `Not Started` or `In Progress`
- If resuming a partially completed phase, verify what's already done before continuing
- Set the plan's frontmatter to `status: active` before touching code (add the
  frontmatter block if an older plan doesn't have one)

### 3. Implement each phase

For each phase:

**Before coding:**
- Read all files that will be modified
- Understand the current state of the code
- Update the plan: change that phase's `Implementation Status:` to `In Progress`
  (the per-phase marker — distinct from the plan-level frontmatter `status:`,
  which is `active` for the whole run)

**Implement the changes:**
- Make the code changes specified in the plan
- Write new tests as specified
- If a phase fails for a non-obvious reason (unexpected test failure, behaviour
  that contradicts the plan's assumptions), do not patch around it. Find the
  root cause first (trace the path, check `git log` on the affected files,
  confirm the cause with a log line or assertion), make the minimal fix, and
  add a regression test that fails without it. Note the cause under the phase.
  If three hypotheses fail, stop and tell the user
- If the plan doesn't match reality (code has changed, approach won't work), STOP:

```
Issue in Phase [N]:
Expected: [what the plan says]
Found: [actual situation]
Proposed adjustment: [how to proceed]

Should I continue with this adjustment?
```

**After coding:**
- Run the contract's `### Checks` **fast tier** (or the single tier when there is no split), honouring its preconditions. Example for a python repository:
  ```
  poetry run ruff check . --fix
  poetry run pyright
  poetry run pytest
  ```
- Fix any failures before proceeding
- Update the plan document:
  - Mark the phase: `Implementation Status: Complete`
  - Add notes about any deviations or learnings under the phase
  - If a decision changed during implementation, update "Decisions Made"

### 4. Between phases

After completing each phase:
- Verify ALL tests still pass (not just the phase's tests); before the phase's commit, that means the **full tier**
- Treat also unrelated failing tests as failures that need to be investigated and probably fixed
- Update the plan with any new insights that affect upcoming phases
- Proceed to the next phase

### 5. After all phases complete

- Run the contract's **full tier** one final time
- Set the plan's frontmatter to `status: done` (leave the file where it is)
- Add a summary section at the bottom:

```markdown
## Implementation Summary

**Completed:** YYYY-MM-DD
**All phases:** Complete
**Test status:** All passing

### Deviations from Plan
- [Any changes made during implementation]

### Learnings
- [Anything discovered that wasn't in the plan]
```

- Present the results to the user

## Guidelines

- **Follow the plan's intent** but adapt to what you find in the code
- **Tests must pass after every phase** - never leave things broken between phases
- **Update the plan as you go** - it should always reflect current reality
- **Stop and ask** when something doesn't match the plan significantly
- **Run all checks** (e.g. ruff, pyright, pytest or similar) after each phase, not just tests
- **Read files fully** before modifying them
- **Commit-worthy phases** - each completed phase should be in a state worthy of committing
- **Park deferred work in `thoughts/todo/`, don't leave it implied by an
  unfinished phase** - the plan closes, the todo survives
