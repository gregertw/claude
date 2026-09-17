# Iterate on Implementation

Iterate on an already-implemented plan: fix bugs, refine UX, add small enhancements, and continuously update the plan document to track what changed and why.

Use this command after a plan has been implemented (via `/implement_plan`) and you are reviewing the result — before, after, or between runs of `/verify_implementation`. Typical items: things you noticed trying it out, wording and flow refinements, small missing pieces, and the issues a verification found. Iterate and verify form a loop; the plan is closed by the latest clean verification.

## Arguments

$ARGUMENTS - Optional: path to the plan file, or description of what to iterate on

## Plan status while iterating

The plan's YAML frontmatter carries its status, from a closed four-value
vocabulary (`proposed` | `active` | `done` | `superseded`).

- **A plan being iterated on stays `done`** (with its `verified:` link intact
  when there is one). Refinements are recorded in the plan's body, not by
  reopening its status — flipping it back to `active` would misreport what's in
  flight.
- **If iteration turns into substantial new work**, that's a new plan via
  `/create_plan`, not an ever-growing Iterations section.
- **Never move the plan file** and never create a `completed/` directory — a
  directory is a *kind* of document, never a *status*. The verification links to
  the plan by path; a move rots that link silently.
- **Findings you're not fixing now** go to `thoughts/todo/<slug>.md` (undated,
  living, deleted when the work lands), not into a phase left open.

## Process

### 0. Project conventions

Before anything else, read the project's `CLAUDE.md` `## Workflow` section and,
if it exists, `.claude/workflow/iterate_plan.md`. They override the defaults in this
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

- If a plan path was provided as argument, read it fully
- If the plan has not been implemented (its status is `proposed` or `active`), stop and tell the user to use `/update_plan` for changes to a plan that has not landed. A verification is not required to iterate
- If not, list recent files in `thoughts/plans/` and ask which plan is being iterated
- Read the plan document and, if the plan has a `verified:` link, that verification document
- Check the verification's "Remaining Tasks" and "Issues Found" sections for known work items
- Run `git diff --stat HEAD` to understand the current state of uncommitted changes

### 2. Understand the iteration scope

Ask the user what needs to change. Iteration typically falls into these categories:

- **Bug fixes**: Something doesn't work as expected
- **UX refinements**: Flow, wording, visual tweaks, interaction details
- **Missing functionality**: Small features not covered by the plan (e.g., a back-link, a loading state)
- **Verification fixes**: Issues identified by a `/verify_implementation` run, when there has been one
- **Integration adjustments**: Components need different props, APIs need different contracts

For each item, confirm with the user:
1. What specifically needs to change?
2. Which files are affected?
3. Is this a deviation from the plan, or within the plan's intent?

### 3. Implement the change

For each change:

**Before coding:**
- Do not go into plan mode
- Read all files that will be modified
- Understand how the change fits with existing code

**Make the change:**
- For **Bug fixes**, follow `/fix_bug` (root cause confirmed before any change,
  minimal diff, regression test that fails before and passes after; it writes
  its own record in `thoughts/research/` and the Iterations entry below);
  for the other categories, implement the fix/refinement directly
- Run relevant tests to verify

**After each change, immediately update the plan document:**

Add or update the **Iterations** section in the plan. If the section doesn't exist, add it after "Implementation Summary" with this structure. Each batch opens with one line of context saying what it followed: the implementation, or a verification of a given date.

```markdown
## Iterations

### YYYY-MM-DD, after implementation (commit `XXXXXX`)
<!-- or: ### YYYY-MM-DD, after verification thoughts/verifications/YYYY-MM-DD-slug.md -->

#### N. [Short title describing the change]

**Category**: [Bug fix | UX refinement | Decision changed | New functionality | Verification fix]

**What changed**: [Specific description of the change]

**Files affected**:
- `path/to/file.tsx` — [what changed in this file]

**Rationale**: [Why this change was needed — what wasn't working, what was discovered]
```

If the section already exists, append a new dated batch; number entries continuously across batches.

### 4. Update related plan sections

If the change modifies a decision, phase behavior, or component contract:

- **Decisions Made**: Add `[Updated YYYY-MM-DD]` annotation to changed decisions, or add new decisions
- **Phase N sections**: Update the "Deviations" subsection with what changed
- **Implementation Summary > Deviations from Plan**: Add the deviation
- **Implementation Summary > Learnings**: Add any new insights

### 5. Run verification after each logical group of changes

After completing a coherent set of changes (e.g., all UX fixes, or all bug fixes), run the contract's `### Checks` fast tier, and the full tier before anything is committed, e.g. for a python code repository:

```bash
poetry run ruff check . --fix
poetry run pyright
poetry run pytest
npm --prefix frontend run build
```

Fix any failures before moving to the next group. For **UX refinements**, also
exercise the changed screens with the **browser** capability
(`~/.claude/tools/browser.md`, via the browser-qa agent): a wording or flow
change that was never loaded in a browser is unverified.

### 6. Summarize iteration progress

After all changes are complete, present to the user:

```
## Iteration Summary

### Changes Made
1. [Change title] — [one-line description]
2. [Change title] — [one-line description]

### Plan Updated
- Iterations section: [N new entries, following implementation / verification of YYYY-MM-DD]
- Decisions updated: [list any]
- Phases affected: [list any]

### Verification Status
- ruff: [pass/fail]
- pyright: [pass/fail]
- pytest: [pass/fail]
- frontend build: [pass/fail]
```

## Guidelines

- **Update the plan after EVERY change, not at the end** — this is the whole point. The plan document should always reflect reality.
- **Small, focused changes** — each iteration item should be independently testable and documentable. Don't batch unrelated changes.
- **Preserve the plan's structure** — add to the Iterations section, don't reorganize existing phases.
- **Re-verify when it matters** — after fixing verification issues or changing behaviour, run `/verify_implementation` again; it writes a new dated document and moves the plan's `verified:` link.
- **Track decisions explicitly** — if you're changing how something works vs what the plan specified, that's a decision change. Document why.
- **Track bugs explicitly** — if you find and fix a bug during iteration, document what the bug was and how it was fixed, not just the fix.
- **Be specific about files** — always list which files were modified and what changed in each.
- **Don't over-document** — a bug fix that changes one line doesn't need a paragraph. Match documentation depth to change significance.
- **Run tests frequently** — don't accumulate 10 changes before running tests. Test after each coherent group.
