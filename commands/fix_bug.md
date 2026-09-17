# Fix Bug

Find the root cause of a bug, fix it with the smallest change that removes the
cause, prove the fix with a regression test, and leave a dated record of what
was wrong. No fix without a confirmed root cause first: a fix that addresses
a symptom makes the next bug harder to find.

## Arguments

$ARGUMENTS - Optional: a description of the bug, an error message, a failing test, or a path to notes

## Where the record goes

The investigation is a dated snapshot in `thoughts/research/YYYY-MM-DD-slug.md`:
research is "what we found out", and a root-cause investigation is exactly that.
Pick a slug that names the symptom (`checkout-double-charge`, not
`fix-payment-service`). No `status:` frontmatter; only plans carry one.

If the bug is in something an implemented plan delivered, also append an entry
to that plan's "Iterations" section, in the format `/iterate_plan` uses, with **Category**: Bug fix and a link to the research
document. The plan stays `done`.

Work you find but are not fixing now goes to `thoughts/todo/<slug>.md`, following
any rule the contract's `### Thoughts` states for that directory.

## Process

### 0. Project conventions

Before anything else, read the project's `CLAUDE.md` `## Workflow` section and,
if it exists, `.claude/workflow/fix_bug.md`. They override the defaults in this
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

### 1. Collect the symptom

- Read whatever was provided: error text, stack trace, failing test, reproduction steps, screenshots
- If context is thin, ask one question at a time until you have: what the user saw, what they expected, and how to trigger it
- Write the symptom down in one sentence before reading any code

### 2. Reproduce

- Trigger the bug deterministically: a failing test, a command, a request, or a browser flow
- If the bug is in a UI, reproduce it in a real browser with the **browser**
  capability (`~/.claude/tools/browser.md`), through the **browser-qa** agent
  for anything beyond a single page load. If no provider is available, say the
  reproduction is by reading only
- If it cannot be reproduced yet, gather more evidence (logs, data state,
  environment differences) before forming a hypothesis. An intermittent bug that
  cannot be triggered is still investigated, but the fix cannot be marked
  verified

### 3. Investigate the root cause

Gather context before forming any hypothesis:

- **Trace the code path** from the symptom back to the candidates: grep every
  reference, read the logic, follow the data
- **Check recent changes**: `git log --oneline -20 -- <affected files>`. If
  this worked before, the root cause is in a diff. With a reproduction in hand
  and a known-good commit, `git bisect` finds the introducing commit
  mechanically; use it rather than guessing across many commits. If the
  suspect is a dependency the contract's `### Dependencies` lists as an
  editable path dependency, bisect it in a worktree of that repository; never
  move its checked-out HEAD
- **Blame the suspect line**: `git blame -L <start>,<end> <file>` tells you
  which change introduced it and what that change was trying to do. Read that
  commit; the intent behind the line often explains the bug
- **Check history in this area**: prior fixes in the same files (`git log`
  on them, `thoughts/research/` for earlier investigations, `thoughts/todo/`
  for known issues). Recurring bugs in the same place are an architectural
  smell, not a coincidence
- **Match against known patterns**:

  | Pattern | Signature | Where to look |
  | --- | --- | --- |
  | Race condition | Intermittent, timing-dependent | Concurrent access to shared state |
  | Null propagation | Type or attribute errors | Missing guards on optional values |
  | State corruption | Inconsistent data, partial updates | Transactions, callbacks, hooks |
  | Integration failure | Timeouts, unexpected responses | External calls, service boundaries |
  | Configuration drift | Works locally, fails elsewhere | Env vars, feature flags, DB state |
  | Stale cache | Old data, fixed by clearing | Caches, CDN, browser, memoization |

- If nothing matches, use the **web-search** capability
  (`~/.claude/tools/web-search.md`, the web-search-researcher agent) on the
  error category (framework plus generic error type), with hostnames, paths,
  and data stripped from the query

Write the result as one line: **Root cause hypothesis:** a specific, testable
claim about what is wrong and why.

### 4. Confirm the hypothesis before changing anything

- Add a temporary log line, assertion, or debug output at the suspected cause
  and run the reproduction. Does the evidence match the claim?
- If it does not, return to step 3. Do not guess. Gather more evidence.
- **Three failed hypotheses: stop.** Tell the user this may be an architectural
  problem rather than a simple bug, and offer: continue with a named new
  hypothesis, get a **second opinion** (the outside-voice agent, given the
  symptom, the evidence, and the rejected hypotheses), escalate to someone who
  knows the system, or add instrumentation and catch it next time.

Red flags, each meaning slow down:
- A fix proposed before the data flow was traced: that is a guess
- "Quick fix for now": there is no "for now"; fix it right or escalate
- Each fix reveals a new problem elsewhere: wrong layer, not wrong code

### 5. Fix

- Lock the scope first with the **scope-lock** capability
  (`~/.claude/tools/scope-lock.md`): the set of paths the fix legitimately
  touches, normally the affected code, its tests, and the changelog. Skip the
  lock only when the cause genuinely spans the repository, and say so
- Fix the cause, not the symptom: the smallest change that eliminates the actual problem
- Minimal diff: fewest files, fewest lines, no refactoring of adjacent code.
  If one guard in a shared function covers every caller, put it there, not in
  each caller
- Remove the temporary instrumentation from step 4
- **More than five files touched: stop** and ask. That is a large blast radius
  for a bug fix. Offer: proceed because the cause genuinely spans them, split
  into the critical path now and the rest as a todo, or rethink the approach

### 6. Prove it

- Write a regression test that **fails without the fix and passes with it**.
  Run it both ways and keep the output. A test that passes on both sides proves
  nothing
- Run the contract's `### Checks` full tier verbatim (lint, types, all tests). No
  regressions allowed
- Reproduce the original scenario fresh, the same way as in step 2, and confirm
  it no longer occurs. This is not optional
- Release the scope lock
- Never say "this should fix it". Either the evidence shows it is fixed, or the
  status below says otherwise

### 7. Write the record

Save to `thoughts/research/YYYY-MM-DD-slug.md`:

```markdown
# Investigation: [Symptom in one line]

**Date:** YYYY-MM-DD
**Branch:** [current branch]
**Commit:** [short hash after the fix]
**Plan:** thoughts/plans/YYYY-MM-DD-slug.md   (when the bug is in something a plan delivered; omit otherwise)

## Symptom

[What was observed, what was expected, how it was triggered.]

## Root Cause

[What was actually wrong and why, with `file:line`. If a change introduced it, name the commit and what it was trying to do.]

## Hypotheses Rejected

- [Hypothesis] - [what evidence ruled it out]

## Fix

- `path/to/file.py:123` - [what changed and why this is the cause, not a symptom]

## Evidence

- Regression test: `tests/path/test_x.py::test_name` - fails at [commit before], passes at [commit after]
- Full suite: [N passed, N failed, N skipped]
- Scope lock: [provider, and the directory]
- Fresh reproduction: [how it was re-run, and the result]

## Related

- [Prior bugs in the same area, todo items, architectural notes. "Recurring in this module: consider ..." when the history says so.]

## Status

DONE | DONE_WITH_CONCERNS | BLOCKED

[DONE: cause found, fix applied, regression test written, all checks pass, fresh reproduction clean.
DONE_WITH_CONCERNS: fixed but not fully verifiable (intermittent, needs staging); say what remains.
BLOCKED: cause unclear after investigation; say what was tried and what is needed.]
```

Then, if the bug belongs to an implemented plan, append the Iterations entry
described above.

### 8. Present the result

Show the user the document path and the status line, the root cause in one
sentence, the files changed, and the regression test. If the status is not
DONE, lead with what is unverified.

## Guidelines

- **No fix without a confirmed root cause** - instrument, reproduce, confirm, then change code
- **Never ship a fix you cannot verify** - if you cannot reproduce and confirm, the status says so
- **Regression test is mandatory** - and it must fail before the fix
- **Minimal diff** - resist the urge to clean up adjacent code; put it in `thoughts/todo/` instead
- **Three failed hypotheses or three failed fixes** - stop and question the architecture, not the next line
- **Blast radius over five files** - ask before proceeding
- **Do not commit** - leave that to the user or `/commit`
- **Do not switch into plan mode** - the investigation record lives in the repository
