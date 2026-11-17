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
  `/create_plan`, not an ever-growing iterations log.
- **Never move the plan file** and never create a `completed/` directory — a
  directory is a *kind* of document, never a *status*. The verification links to
  the plan by path; a move rots that link silently.
- **Findings you're not fixing now** go to `thoughts/todo/<slug>.md` (undated,
  living, deleted when the work lands), not into a phase left open.
- **Iterations go in the plan's iterations log**, a sibling file
  (`thoughts/plans/YYYY-MM-DD-slug.iterations.md`, format in
  `~/.claude/shared/conventions.md`), linked from the plan's `iterations:`
  frontmatter. The plan itself stays the design, readable in one sitting.

## Process

### 0. Project conventions

Read `~/.claude/shared/conventions.md` and follow its Project conventions
section; this command's addendum is `.claude/workflow/iterate_plan.md`. The rest of
that file defines the document layout, plan frontmatter, acceptance
criteria, living specs and the iterations log this command refers to.

### 1. Load context

- If a plan path was provided, read it; otherwise list recent files in `thoughts/plans/` and ask which plan. If its status is `proposed`, stop and point to `/update_plan`; if `active`, point to `/implement_plan`, which resumes it. A verification is not required to iterate
- Read the plan document, its iterations log if it has one, and, if the plan has a `verified:` link, that verification document
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
  its own record in `thoughts/research/` and the iterations log entry below);
  for the other categories, implement the fix/refinement directly
- Run relevant tests to verify

**After each change, immediately append to the iterations log:**

- Create `thoughts/plans/YYYY-MM-DD-slug.iterations.md` on the first entry
  (the plan's date and slug), and add `iterations:` to the plan's frontmatter.
  A plan with an older in-plan `## Iterations` section keeps it, gets a
  one-line pointer to the log under it, and new entries go to the log
- Each batch opens with a dated heading saying what it followed: the
  implementation (with commit), or a verification (with its path)
- One entry per change, one to three lines, numbered continuously across
  batches:

```markdown
1. **[Category] Short title** — what changed (`path/a.py`, `path/b.tsx`). Why: [one line]. AC: [AC-n added / changed / none]. Spec: [updated / delta amended / none].
```

Categories: Bug fix, UX refinement, Decision changed, New functionality,
Verification fix.

**Behaviour changes carry their AC and spec with them.** A Decision changed,
New functionality, or UX refinement that changes what a user can observe:

- adds or changes an acceptance criterion in the plan (new ones continue the
  numbering and say `(added in iteration N)`), with a test, seen failing
  first as in `/implement_plan`, and a row in the Traceability table
- updates the spec: without `spec_merged:` on the plan, amend its Spec Delta;
  with it, edit the living spec directly, citing the iterations entry, and set
  its `**Last changed:**` (rules in `~/.claude/shared/conventions.md`,
  "Living specs")

### 4. Update related plan sections

If the change modifies a decision, phase behavior, or component contract:

- **Decisions Made**: Add `[Updated YYYY-MM-DD]` annotation to changed decisions, or add new decisions
- **Phase N sections**: Update the "Deviations" subsection with what changed
- **Implementation Summary > Deviations from Plan**: Add the deviation
- **Implementation Summary > Learnings**: Add any new insights

### 5. Run verification after each logical group of changes

After completing a coherent set of changes (e.g., all UX fixes, or all bug fixes), run the contract's `### Checks` fast tier verbatim, and the full tier before anything is committed.

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
- Iterations log: [N new entries, following implementation / verification of YYYY-MM-DD]
- Acceptance criteria: [added / changed, or none]
- Spec: [delta amended / spec updated / none]
- Decisions updated: [list any]
- Phases affected: [list any]

### Verification Status
- [each command in the tier]: [pass/fail, counts]
```

## Guidelines

- **Log after EVERY change, not at the end** — this is the whole point. The plan, its log, and the spec should always reflect reality.
- **Small, focused changes** — each iteration item should be independently testable and documentable. Don't batch unrelated changes.
- **Preserve the plan's structure** — iterations go to the log; the plan body changes only where a decision, AC, or the Spec Delta changed.
- **A behaviour change without an AC is invisible to the next verification** — add the AC and its test.
- **Re-verify when it matters** — after fixing verification issues or changing behaviour, run `/verify_implementation` again; it writes a new dated document and moves the plan's `verified:` link.
- **Track decisions explicitly** — if you're changing how something works vs what the plan specified, that's a decision change. Document why.
- **Track bugs explicitly** — if you find and fix a bug during iteration, document what the bug was and how it was fixed, not just the fix.
- **Be specific about files** — always list which files were modified and what changed in each.
- **Don't over-document** — an entry is one to three lines; the investigation record (for bugs) or the commit carries the rest.
- **Run tests frequently** — don't accumulate 10 changes before running tests. Test after each coherent group.
