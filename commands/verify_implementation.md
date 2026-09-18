# Verify Implementation

Review the implementation against the plan, research, and feature documents. Verify correctness, test coverage, and whether the feature's success measures can be observed, then create a verification document with findings.

## Where the document goes, and what it does to the plan

The verification goes in `thoughts/verifications/YYYY-MM-DD-slug.md`, **reusing
the plan's slug** — same slug across feature → research → plan → verification is how a
piece of work is followed end to end. The date is today's, not the plan's.

The document is a dated snapshot: it records what was true when it was written
and is not edited afterwards. It carries **no `status:` frontmatter** — only
plans do.

**Verification is re-runnable.** Iterating and verifying form a loop: run this
again after `/iterate_plan` has addressed findings or changed behaviour. Each
run writes a new dated document; the previous ones stay as history. On a
re-run, read the previous verification first, check whether each of its Issues
Found and Remaining Tasks was addressed, and say so in the new document. The
plan's `verified:` link always points at the newest verification.

Two things this command owns on the plan side:

- **Write the back-link.** When the verification confirms the plan landed, set
  the plan's frontmatter to `status: done` and add
  `verified: thoughts/verifications/YYYY-MM-DD-slug.md`.
- **Only link what you actually verified.** `verified:` asserts that *this plan*
  was verified. If the verification covers a subset — two follow-up items, one
  phase of six — say so in the document and leave the plan at bare
  `status: done`; the verification's own `**Plan:**` line keeps the thread
  discoverable. An over-claimed `verified:` reads as proof later.

**Never move the plan** on the strength of a verification, and never create a
`completed/` directory: a directory is a *kind* of document, never a *status*.
The `**Plan:**` path you write below is exactly the link a move would rot.

## Process

### 0. Project conventions

Before anything else, read the project's `CLAUDE.md` `## Workflow` section and,
if it exists, `.claude/workflow/verify_implementation.md`. They override the defaults in this
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

### 1. Gather context

- If a plan path was provided as argument, read it fully
- If not, list recent files in `thoughts/plans/` and ask which to verify
- Read the referenced research document
- If the plan or research has a `**Feature:**` line, read that feature document
  fully: its Desired Outcome, User Experience, and Success Measures are what the
  implementation is ultimately measured against
- Read the plan, noting which phases are marked complete, and its Iterations
  section if it has one
- If the plan already has a `verified:` link, read that verification: this run
  must report the fate of every issue it found (fixed, still open, accepted)

### 2. Run all automated checks

Run the contract's `### Checks` **full tier** verbatim, including any preconditions it names. Example for a python repository:
```
poetry run ruff check .
poetry run pyright
poetry run pytest
```

Record the results. If anything fails, note it but continue the review.

### 3. Verify each phase

For each phase marked as complete in the plan:

- Read every file listed in the phase's "Changes" section
- Verify the changes match what was planned
- Check that the tests listed in "New Tests" exist and are meaningful
- For each entry under the phase's "Failure Scenarios", confirm the test or
  handling the plan promised actually exists, and cite `file:line`
- Run the phase-specific verification commands
- Note any deviations (acceptable or concerning)

Use **Explore** or **codebase-analyzer** sub-agents for parallel investigation when reviewing multiple independent areas.

### 4. Check the feature's success measures

Skip this step only when there is no feature document.

For each row in the feature document's Success Measures table:

- **Is it observable now?** Find the log line, metric, event, query, or manual
  check the feature document said would be used. Confirm it exists in the code
  and fires on the paths the feature exercises. Cite `file:line`.
- **Does it measure the right thing?** A counter that exists but counts the
  wrong event, or fires before the outcome is actually achieved, is not
  observable. Say so.
- **Can the target be checked before shipping?** Some measures only accumulate
  after release (support tickets, adoption). Mark those "post-ship" and record
  what the baseline is today, so the comparison is possible later. Never mark a
  post-ship measure as met.
- **Did the plan drop a measure?** If a measure has no observation mechanism and
  the plan never addressed it, that is an issue, not a deviation.

Then walk the feature's User Experience main path and edge paths against the
implementation: does each step exist, and does the edge path end the way the
feature document said it would? For any path with a UI, do this walk in a real
browser (step 4.5), not by reading templates.

### 4.5. Exercise the UI in a browser

This step runs whether or not there is a feature document. Skip it only when no
phase touches anything a user sees. A UI change that was never loaded in a
browser is unverified, whatever the tests say.

**What to test.** Derive the pages from the plan, not from a diff: each phase's
"Changes" list names the routes, views, and components it touched, and the
feature document's User Experience names the paths a user takes through them.
For each page:

- Load it and read the console before doing anything; then check the console
  again after every interaction. An error or warning that appears on a path the
  feature exercises is a finding.
- Click every interactive element the phase added or changed. Does it do what
  its label says?
- Submit every form the phase touched three ways: empty, invalid, and with edge
  input (very long text, special characters). Does validation fire, and does a
  valid submit end where the feature document says it ends?
- Exercise the states: empty (nothing to show), loading, error, and overflow.
- Check the page at a mobile width (375 px wide) if users reach it on a phone.
- Walk the feature's main path end to end, and each edge path to where the
  feature document says it ends.

Severity for browser findings: **critical** blocks a core workflow, loses data,
or crashes; **high** breaks a major feature with no workaround; **medium** works
with a noticeable problem and a workaround exists; **low** is cosmetic. Every
finding gets reproduction steps and one screenshot. Save screenshots outside
the repository (for example `/tmp/qa-<slug>/`); the verification document
describes each finding in text and is the durable record.

**Which tool.** Use the **browser** capability (`~/.claude/tools/browser.md`):
first available provider wins, a `### Tools` pin in the project's `CLAUDE.md`
wins over that. Delegate the whole walk to the **browser-qa** agent
(`~/.claude/agents/browser-qa.md`): give it the base URL, the pages from the
phases' Changes lists, the paths from the feature document, and a screenshot
directory outside the repository. It picks the provider, reports which one it
used, and returns findings in the severity vocabulary above. The app must
already be running; the contract's `### App` says how to start it and which
hosts are allowed.

If the agent reports `Browser QA: tool unavailable`, record that in the
document, walk the UI paths by reading templates and components, and say
plainly in the Overall Assessment that the UI was not exercised.

### 5. Check for gaps

**Scope, both directions.** Nothing less: were any planned items skipped without
explanation? Nothing more: does the diff touch files, add features, or refactor
code that no phase asked for? "While I was in there" changes are findings, even
when they are improvements, because nobody reviewed them against a plan.

**Defect classes.** Read the changed code with each of these in mind. Every
finding quotes the line, as `file:line` plus the text; a concern that cannot
quote a line is a note, not a finding.

- **Data safety and injection**: SQL built by interpolation; shell commands
  built from input; HTML rendered unescaped from user or model output; direct
  writes that bypass model validation.
- **Concurrency**: read-check-write without a unique constraint or atomic
  update; find-or-create without a unique index; non-atomic status transitions;
  sync calls inside async code.
- **Value completeness**: a new enum value, status, tier, or constant traced
  through *every* consumer, including files outside the diff: every switch,
  filter, display, and allowlist that enumerates its siblings. This is the one
  class where reading only the diff is not enough.
- **Trust boundaries**: values produced by an LLM or an external service that
  are stored, sent, or fetched without validating shape, format, or an allowlist.
- **Conditional side effects**: a branch that forgets a side effect its sibling
  performs; a log line that claims an action that was skipped; an event emitted
  only on the happy path.
- **Error and edge handling**: empty, zero, null, maximum, single-element,
  unicode, double-submit, navigate-away mid-operation, stale data. Are these
  handled, and are the error paths tested?
- **Migrations and contracts**, when the phase touches them: reversible;
  no data loss (dropped columns, narrowed types, NOT NULL without backfill);
  safe with old code running against the new schema; API changes that remove
  or rename fields, add required parameters, or change status codes.
- **Hygiene**: leftover TODOs, debug code, excessive logging, commented-out
  code, dead code, stale comments and docstrings.

**Adversarial pass.** Use the **second-opinion** capability
(`~/.claude/tools/second-opinion.md`) through the **outside-voice** agent
(`~/.claude/agents/outside-voice.md`): give it the diff range, the plan path,
and the findings so far, with this brief: think like an attacker and a chaos
engineer. Find the ways this code fails in production: edge cases, races,
security holes, resource leaks, silent data corruption, wrong results with no
error, swallowed exceptions, trust boundary violations. The agent runs a second
model when one is installed and is the fresh-context reviewer otherwise; it
says which. Because the target here is a diff, also run the capability's
skill providers yourself, in this context, when they are listed: the built-in
code review at a high effort level without `--fix`, and the built-in security
review when the diff touches auth, input handling, data access, or anything
that leaves the process. A `run <script>` pin replaces only the external-model
providers; these two still run. A `none` pin skips all of it. Fold everything into Issues Found with the same
evidence standard, and record each provider under Automated Check Results.

### 6. Write the verification document

Save to `thoughts/verifications/YYYY-MM-DD-slug.md`:

```markdown
# Verification: [Feature Name]

**Date:** YYYY-MM-DD
**Feature:** thoughts/features/YYYY-MM-DD-slug.md   (when there is one; omit otherwise)
**Plan:** thoughts/plans/YYYY-MM-DD-slug.md
**Research:** thoughts/research/YYYY-MM-DD-slug.md
**Branch:** [current branch]
**Commit:** [current short hash]

## Automated Check Results

For each check, specify the check, whether it passed or failed and details if fail. Examples for a python repository:

- **Browser QA:** [Ran, provider: X / N/A (no UI phases) / Skipped by project policy / Tool unavailable - findings, if any, by severity]
- **Second opinion:** [provider: X / Skipped by project policy - findings, if any]
- **Ruff:** [Pass/Fail - details if fail]
- **Pyright:** [Pass/Fail - details if fail]
- **Pytest:** [Pass/Fail - N passed, N failed, N skipped]

## Phase Verification

### Phase 1: [Name] - [VERIFIED / ISSUES FOUND]

**Changes verified:**
- `file.py:123` - [matches plan / deviation noted]

**Tests verified:**
- [test name] - [passes / meaningful coverage]

**Deviations from plan:**
- [Any differences and whether they're acceptable]

### Phase 2: [Name] - [VERIFIED / ISSUES FOUND]

...

## Success Measures

(Omit when there is no feature document.)

| Measure | Target | Observable? | Evidence | Status |
| --- | --- | --- | --- | --- |
| [from feature doc] | [from feature doc] | Yes / No / Partly | `file.py:123` - [what fires, when] | Met / Not met / Post-ship (baseline: ...) / Not observable |
| [...] | [...] | [...] | [...] | [...] |

**User experience walk-through:**
- **Main path** - [each step present / step N missing or differs: how]
- **[Edge path]** - [ends as described / differs: how]

## Previous Verification

(Omit on a first run.)

**Previous:** thoughts/verifications/YYYY-MM-DD-slug.md

| Issue then | Status now | Evidence |
| --- | --- | --- |
| [Issue 1 from the previous run] | Fixed / Still open / Accepted as-is | `file.py:123` or the Iterations entry |

## Remaining Tasks

- [ ] [Anything not yet completed from the plan]
- [ ] [Any follow-up work identified during verification]

## Issues Found

### [Issue 1]
**Severity:** [High/Medium/Low]
**Description:** [What's wrong]
**Location:** `file.py:line`
**Recommendation:** [How to fix]

## Overall Assessment

[1-2 paragraph summary: Is the implementation complete and correct? When there
is a feature document: does it deliver the Desired Outcome, and will we be able
to tell? What needs attention before merging?]
```

### 7. Update the plan

- Set the plan's frontmatter `status: done` and set `verified:` to the
  document you just wrote (replacing an older link: the newest verification is
  the one that counts) — unless the verification only covers a subset of the
  plan, in which case leave `verified:` as it was and say why in the document
- Leave the plan file where it is
- Anything in "Remaining Tasks" that nobody is picking up now belongs in
  `thoughts/todo/<slug>.md`, following any rule the contract's `### Thoughts`
  states for that directory, so the plan can close instead of lingering

### 8. Present findings

Show the user a summary of the verification with the document path,
highlighting any issues that need attention and any success measure that is not
observable.

## Guidelines

- **Read the actual code** - don't rely on the plan's claims of completion
- **Run all checks yourself** - verify automated results firsthand
- **Be specific** about issues - include file paths, line numbers, and concrete descriptions
- **Distinguish severity** - not every deviation is a problem
- **Check test quality** - tests that exist but don't test the right things are worse than no tests
- **Report, don't fix** - this command writes findings; fixes belong to
  `/iterate_plan` and `/fix_bug`, where they get their own regression tests and
  plan entries
- **A UI nobody loaded is unverified** - say so in the assessment rather than
  letting passing tests imply it works
- **A measure nobody can observe is an issue** - the feature document promised
  a way to know whether this worked; if the implementation didn't provide it,
  say so with the same severity as a missing test
- **Never mark a post-ship measure as met** - record the baseline and leave the
  verdict for after release
