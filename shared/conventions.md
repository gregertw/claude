# Workflow conventions

Shared rules the workflow commands read at their step 0. A command's own text
adds to these; it never repeats them. The project's contract and addenda
override both.

## Project conventions

Before anything else, read the project's `CLAUDE.md` `## Workflow` section and,
if it exists, `.claude/workflow/<command>.md`. They override the defaults in
the command. They cover:

- the checks to run (fast and full tiers). The preconditions and warnings
  listed with them are instructions.
- how the app starts
- the CI policy
- tool pins
- any rule under `### Thoughts` about writing there
- any rule the project adds

If neither exists:

- Proceed on the command's defaults.
- Mention once that `bash ~/.claude/bin/init-project` sets the project up.
- Do not invent project facts (check commands, hosts, credentials). Ask for
  them, or leave them as open items.

When running autonomously with no user to ask, do not block. Derive what you
can from the documents and mark every such decision "deduced, not confirmed".
List it under the command's undecided or deferred section. Never record an
approval (see below) on the user's behalf.

## Where documents live

The project's `thoughts/README.md` defines the layout of `thoughts/` and is
authoritative. Its default, `~/.claude/templates/thoughts-README.md`, has:

- **dated snapshots:** `features/`, `research/`, `plans/`, `verifications/`
- **living documents:** `specs/`, `reference/`, `todo/`

**A directory is a *kind* of document, never a *status*.**

If a command needs a directory the README does not list, ask before creating
it, and add the row to the README when the user agrees. When running
autonomously, stop and report instead.

Never move a document because its status changed, and never create a
`completed/` directory. Moving a document rots every link that other documents
wrote to it.

- **Dated** (`YYYY-MM-DD-slug.md`) means *snapshot*: true as of that date, and
  not edited afterwards except to correct an error. Two exceptions: the body
  of a plan, which tracks reality while the plan is implemented, and its
  append-only iterations log.
- **Listing plans.** When a command lists `thoughts/plans/` to offer a choice,
  it ignores `*.iterations.md`: those are logs, not plans.
- **Undated** (`slug.md`) means *living*: edited in place, and deleted when it
  stops being true.
- **Reuse the slug** across feature → research → plan → verification. Same
  slug means the same thread of work: the dates differ, the slug does not.
  `/plan_feature` mints it, or the first command in the thread when there is
  no feature document. A slug names the outcome, not the implementation
  (`offline-sync`, not `sqlite-queue`).

## Plan frontmatter

Only plans carry frontmatter state. Research, feature and verification
documents are dated snapshots, and their date is their status.

```yaml
---
status: proposed | active | done | superseded
approved: YYYY-MM-DD by <who>                         # set by /create_plan or /update_plan on the user's explicit approval
verified: thoughts/verifications/YYYY-MM-DD-slug.md   # set by /verify_implementation
spec_merged: YYYY-MM-DD                              # set by /verify_implementation when it merges the Spec Delta
superseded_by: thoughts/plans/YYYY-MM-DD-slug.md      # when status: superseded
iterations: thoughts/plans/YYYY-MM-DD-slug.iterations.md  # once the first iteration is logged
---
```

`status` has four values, and nothing else is valid:

- **proposed:** written, and nobody is working on it.
- **active:** being implemented right now.
- **done:** implemented.
- **superseded:** overtaken. Keep the file: what we decided *not* to do is
  worth as much as what we did.

`grep -l "^status: active" thoughts/plans/*.md` must answer "what is in flight"
truthfully.

`approved:` is the gate between writing a plan and building it:

- `/implement_plan` does not start a plan without it.
- A substantive `/update_plan` change removes it, so the change gets approved
  too.
- Plans written before this convention are treated as approved when they are
  already `active` or `done`.

The frontmatter `status:` describes the whole plan. Each phase also has its own
`Implementation Status:` line (`Not Started` / `In Progress` / `Complete`).
Keep the two separate.

## Acceptance criteria

An acceptance criterion (AC) is a pass/fail statement about behaviour that can
be observed from outside the code: in the UI, in an API response, in stored
state, or in a log line. Write it as:

```
- **AC-3** (edge: action fails) — When the upload fails, the user sees the
  file name and a Retry button, and nothing is saved.
```

- **Where they come from.** `/plan_feature` writes ACs from the User
  Experience: at least one per main-path step that changes behaviour, and one
  per edge path. Without a feature document, `/create_plan` writes them.
- **IDs are stable.** They are numbered once and never renumbered or reused. A
  dropped AC stays in the plan's list, struck through, with the reason.
- **ACs added later continue the numbering.** One added by the plan or by an
  iteration says where it came from: `(added in plan: reason)` or
  `(added in iteration N)`.
- **One behaviour per AC.** If an AC has "and" joining two independent
  results, split it.
- **ACs are not Success Measures.** An AC is checked before shipping. A
  Success Measure tells whether the outcome happened, often after shipping.

**Traceability.** Every plan has a table that maps each AC to the phase that
delivers it and the test that proves it:

```
| AC | Phase | Test | Status |
| --- | --- | --- | --- |
| AC-1 | 1 | `tests/test_share.py::test_owner_can_share` | Not run / Failing (red) / Passing |
| AC-4 | 2 | manual: browser walk, edge path "no recipients" (why no automated test) | ... |
```

An AC with no test and no named manual check is a gap in the plan, not a
detail.

## Living specs

`thoughts/specs/` is undated and living. It describes how each capability
behaves **today**, in the language of the feature documents, not the code.
**Specs are for the repository's developers and maintainers.** User-facing
documentation (a docs tree, help pages, a README written for users) says what
users are told: it is a *source* for a spec, and a disagreement between the
two is a finding, but it never stands in for one. The two describe the same
behaviour for different readers, so they are kept apart on purpose. The
contract's `### Thoughts` may move the spec directory to another
maintainer-facing location; these rules apply there. `thoughts/specs/README.md` is the index: one row per capability with
its file, its status (`draft` / `reviewed` / `partial`), and the date it
was last reviewed. `/create_spec` writes the index and backfills specs.

**Granularity is set by what the spec describes, not by its length.**

- **In:** what a user, an API client, or an agent can observe: inputs,
  results, errors, limits, permissions, states, and the rules that decide
  between them.
- **Out:** how the system does it. That means classes, tables, and call
  order. A statement that would change during a refactor that nobody notices
  does not belong.
- **One capability is what a user would name**: "sharing", "memory search",
  "the wiki". It is not a code module.
- **Split a capability into a directory** (`specs/sharing/README.md` plus one
  file per area) when either holds:
  - its areas change independently, meaning plans tend to touch one area and
    not the others
  - one file passes about 600 lines

  `README.md` keeps the summary and the decisions that span areas.
- **Merge two specs** when every plan touches both.

```markdown
# Spec: [Capability]

**Status:** draft | reviewed | partial (covers: [what])
**Reviewed:** YYYY-MM-DD by <who>
**Last changed:** YYYY-MM-DD by thoughts/plans/YYYY-MM-DD-slug.md (or /create_spec)
**ID prefix:** SHR

## Summary
[What this capability is for, for whom, and its main rules: one screen at
most. A reader who stops here knows what it does and what it never does.]

## [Area, e.g. Inviting people]

- **SHR-1** — When [trigger], [observable result]. [test] `tests/test_share.py::test_invite`
- **SHR-2** — When [trigger], [observable result]. [doc+code]
- **SHR-3** — When [trigger], [observable result]. [plan] plans/2026-10-17-share-output-document.md

## Decisions in force
- **SHR-D1** — [decision users or later plans must respect, and why] (plans/YYYY-MM-DD-slug.md)

## Not supported
- [Thing users might expect] — [why]

## Untested
- SHR-2, SHR-3 — no test asserts them
```

- **IDs are stable.** One prefix per capability. IDs are never renumbered or
  reused, and a removed statement's ID is retired. Spec Deltas and acceptance
  criteria refer to statements by ID.
- **Source tags** say what supports the statement: `[test]`, `[doc]`,
  `[plan]` and `[code]`, combined with `+`. A tag with `test` names the test.
  `[code]` alone never reaches a `reviewed` spec: the human confirms it as
  intended (its tag becomes `[code, confirmed]`) or it becomes a bug.
- **Other checks.** A statement may also name a project check that is not a
  test, after its tests: an audit brief, a property check, a lint rule
  (`audit scripts/audits/x.md`). It supports the statement but does not make
  it tested: a statement with no test stays under Untested, marked as
  checked. Where a project keeps such checks is named in its addendum.
- **Status:**
  - `draft` is not yet reviewed by a human. Plans may build on it, but they
    treat each statement as a claim to check.
  - `reviewed` is confirmed by a human.
  - `partial` covers only what is listed in its `Status:` line, so a missing
    statement does not mean "not supported".

**Delta, then merge.** Plans never rewrite a spec directly. Each plan carries
a `## Spec Delta` section per spec file:

- ADDED statements, which get the next free ID at merge
- MODIFIED and REMOVED statements, by ID
- DECISION lines

`/verify_implementation` merges the delta into the spec when a whole-plan
verification has no open High issue, and records that with `spec_merged:` in
the plan's frontmatter. `spec_merged:` is the only signal the other commands
use:

- **No `spec_merged:` yet:** iterations and bug fixes that change behaviour
  amend the plan's Spec Delta; the merge picks them up.
- **`spec_merged:` present:** they edit the living spec directly, citing the
  iterations entry or the investigation, and set its `**Last changed:**`.
- **A new or changed check is a spec change.** When work writes or changes a
  check that states a rule (an audit brief written because review rounds
  kept finding the same kind of bug, say), its rule goes into the Spec Delta,
  or into the spec directly once `spec_merged:` is set, and its accepted
  exceptions become decisions or Not supported entries.
- **A plan with no Spec Delta** (written before this convention) merges
  nothing. Later changes to it edit the spec only when a spec exists, and are
  otherwise noted in the iterations log.

**A capability with no spec.** When `/create_plan` touches one, it offers
`/create_spec <capability>` first, so the delta has a baseline. If the user
declines, or the run is autonomous, the merge creates a `partial` spec from
the delta alone.

## Iterations log

Refinements after implementation go in a sibling file, so the plan stays a
readable design:

- File: `thoughts/plans/YYYY-MM-DD-slug.iterations.md`, using the plan's date
  and slug.
- Linked from the plan's `iterations:` frontmatter.
- Append-only.

A plan written before this convention that already has an `## Iterations`
section keeps it. New entries still go to the sibling file, and the old
section gets a one-line pointer to it.

```markdown
# Iterations: [Plan name]

**Plan:** thoughts/plans/YYYY-MM-DD-slug.md

## YYYY-MM-DD, after implementation (commit `abc1234`)
<!-- or: after verification thoughts/verifications/YYYY-MM-DD-slug.md -->

1. **[Category] Short title** — what changed (`path/a.py`, `path/b.tsx`). Why: [one line]. AC: [AC-n added / changed / none]. Spec: [updated / delta amended / none].
```

Categories: Bug fix, UX refinement, Decision changed, New functionality,
Verification fix.

- One entry per change, numbered continuously across batches.
- An entry takes one to three lines. A change that needs more than that
  belongs in the investigation document or a new plan.
