# Create Spec

Build living specs for a repository that already has code. These describe what
each capability does today and what it is meant to do. The command has two
modes:

- **No argument: inventory.** Propose the map of capabilities, agree it with
  the user, and write the index `thoughts/specs/README.md`. Write no spec
  bodies in this mode.
- **`<capability>`: backfill one spec.** Gather the behaviour from tests,
  docs, plans and code, draft the spec, review it with the user, and route
  what the review turns up. Pass a capability with `partial` status to
  complete it.

Specs are built when work needs them. Start with the capabilities that plans
are about to touch: `/create_plan` offers this command when it finds none. A
capability nobody is changing may never need a spec.

## Arguments

$ARGUMENTS - Optional: a capability name from the index, or `--refresh`. With
`--refresh`, re-run the inventory against an existing index: add newly
appeared capabilities, flag stale ones, and leave confirmed rows alone.

## What a spec is

The format, granularity, IDs, source tags, statuses and split rule are in
`~/.claude/shared/conventions.md` ("Living specs"). Two rules matter most here:

- **A spec states intended behaviour, not a transcript of the code.** Code
  shows what happens, which includes today's bugs. A statement supported only
  by code is a question for the human, never a requirement on its own.
- **Granularity is what an outsider can observe.** That covers a user, an
  API client, or an agent. Detail that a refactor could change without anyone
  noticing stays out, however long or short that leaves the spec.

## Process

### 0. Project conventions

Read `~/.claude/shared/conventions.md` and follow its Project conventions
section; this command's addendum is `.claude/workflow/create_spec.md`. The rest of
that file defines the document layout, plan frontmatter, acceptance
criteria, living specs and the iterations log this command refers to.

Find the spec location: `thoughts/specs/`, or the location the contract's
`### Thoughts` names. If the project's `thoughts/README.md` does not list
`specs/`, ask before creating it.

## Inventory mode

### I-1. Gather candidates

Spawn **Explore** agents in parallel. Each sweeps one kind of source and
returns user-nameable capabilities with evidence:

- **Entry points:** HTTP routes, API resources, CLI commands, agent or MCP
  tools, scheduled jobs, webhooks
- **The user surface:** app navigation, pages, settings sections, help pages
- **Docs:** the user-facing documentation, wherever the project keeps it.
  Projects name and lay it out differently; use the location `CLAUDE.md` or the
  contract names, and otherwise find it and confirm it with the user in step
  I-2. Help pages and a user-facing README count too. They name
  capabilities the way users know them; they are a source, never a spec (see
  "Living specs")
- **History:** slugs of `thoughts/features/` and `thoughts/plans/`, grouped by
  topic; `CHANGELOG.md` headings

Ask each agent for names from the user's side ("sharing"), not module names
(`share_service`).

### I-2. Propose the map

Merge the candidates into capabilities. One capability is what a user would
name. Use the history as the tiebreaker:

- Areas that plans always change together belong in one spec.
- Areas that plans change separately are separate specs, or areas inside a
  split spec.

Present it as a table:

```
| Capability | ID prefix | Main sources | Plans touching it (latest) | Size | User-facing docs | Priority |
| --- | --- | --- | --- | --- | --- | --- |
| Sharing | SHR | [entry points, UI components] | 6 (2026-11-11) | L: split? | [user doc pages, or none] | 1: active work |
```

- **Size** is a rough count of behaviours: S (under about 20), M (about
  20-60), L (over about 60). An L should be split into areas.
- **Priority** follows upcoming and recent work, then user impact.

Ask the user to confirm the boundaries, the names, the ID prefixes, and the
order. Work in rounds; do not
present a final list and ask for a yes.

### I-3. Write the index

Write `thoughts/specs/README.md`:

```markdown
# Specs

How each capability behaves today. Format and rules: ~/.claude/shared/conventions.md ("Living specs").

| Capability | File | Prefix | Status | Reviewed | Priority |
| --- | --- | --- | --- | --- | --- |
| Sharing | sharing/README.md | SHR | none | - | 1 |
| Memory search | memory-search.md | MEM | none | - | 2 |
```

Present it, and point to the next step:
`/create_spec <highest-priority capability>`.

## Backfill mode

### B-1. Load

- Read the index row for the capability. With no index, run the inventory
  first, or ask the user to name the capability's boundary and ID prefix.
- Read the existing spec, if it is `partial`: its statements and IDs stay.
- Read every feature document, plan and verification whose slug or overview
  belongs to the capability. List them, and show the user the list before
  spawning agents.

### B-2. Gather, one agent per source

Spawn these in parallel. Give each one the capability boundary from the index
and the rule above about what a spec contains. Each returns candidate
statements in the form "When [trigger], [observable result]", each with
`file:line`:

- **Tests** (codebase-analyzer): what the tests assert about this capability.
  Name the test for each statement. Only assertions count; a test that merely
  calls the code does not.
  Also read the project's other checks that the addendum names (audit
  briefs and the like): each rule they state is a candidate statement, cited
  with the check, and their accepted exceptions are candidate decisions or
  Not supported entries.
- **Docs** (Explore): what the user-facing docs, help pages, and agent-facing
  text (tool descriptions, guides) promise.
- **Plans** (Explore, over `thoughts/`):
  - Decisions Made
  - Not In Scope, which feeds "Not supported"
  - every "Decision changed" iteration, which wins over the original decision
  - feature-document Acceptance Criteria, where they exist

  Read the latest plan as authoritative where plans disagree.
- **Code** (codebase-analyzer): what the entry points from the index do.
  Cover inputs, results, errors, limits, permissions and states. Include only
  behaviour visible from outside.

### B-3. Reconcile into a draft

- **Merge.** Merge statements that say the same thing into one and combine
  their source tags (`[test+doc+code]`).
- **Find discrepancies.** A discrepancy is two sources that disagree: a doc
  promises X and the code does Y, a plan decided X and a test asserts Y, or a
  test asserts something no code path reaches. Do not resolve any of these.
  Collect them.
- **Find code-only statements.** These are statements tagged only `[code]`.
  Collect them for review.
- **Group and number.** Group statements into areas, in the order a user
  meets them, and number them with the prefix. In a `partial` spec, keep
  existing IDs and continue the numbering.
- **Size check.** If the draft passes about 600 lines, or its areas map to
  plans that never overlap, propose a split into a directory before writing.

Write the draft with `**Status:** draft`. The review list stays in the
conversation, not in the spec.

### B-4. Review with the user

Go through it in this order, a few items per round:

1. **Discrepancies.** For each one, show both sources with `file:line` and ask
   which is intended. The answer goes into the spec. If the code is the side
   that is wrong, it is a bug, and the spec states the intended behaviour.
2. **Code-only statements.** Ask "Is this intended?", grouped by area. If
   yes, the tag becomes `[code, confirmed]`. If no, it is a bug. If it is
   unimportant, the statement is removed: it is implementation detail.
3. **The summary and Not supported.** Ask whether a user reading only the
   Summary would be surprised by anything in it.

When the user has been through everything, set
`**Status:** reviewed` and `**Reviewed:** YYYY-MM-DD by <who>`. If the user
stops partway, the spec stays `draft` and its unreviewed items stay listed.
When running autonomously, the spec stays `draft`, and the discrepancies go
to a todo for a human.

### B-5. Route what the review found

- **Bugs:** each becomes a `thoughts/todo/<slug>.md` entry, or goes straight
  to `/fix_bug` if the user wants. Follow any rule in the contract's
  `### Thoughts` for that directory. The spec states the intended behaviour
  and lists the bug's todo next to the statement.
- **Untested statements:** list them in the spec's Untested section. If there
  are many, write one todo for the capability's test gaps.
- **Stale docs:** when a doc turned out wrong, add a todo naming the page and
  the statement. Do not edit product docs from this command.
- **The index:** update the row's status, reviewed date and file.

### B-6. Present

Show the user:

- the spec path, its status, and the number of statements
- how many statements came from each source, how many were code-only and
  confirmed, and how many discrepancies there were and what each became
- the todos written

Then point to the next capability in the index.

## Guidelines

- **Don't write the code down as the spec.** Every statement supported only
  by code is a question, and a `reviewed` spec has none left unanswered.
- **Never resolve a discrepancy silently.** It is the most valuable thing this
  command finds.
- **Do one capability per run.** The review is the expensive part. Ten specs
  at once means ten specs that nobody really reviewed.
- **User-facing docs are a source, not the spec** - the spec is written for
  maintainers; where docs and spec disagree, that is a discrepancy to review,
  and a doc that is wrong gets a todo
- **Leave the code alone.** Bugs go to todos or `/fix_bug`. This command
  writes specs, the index and todos, nothing else.
- **Do not switch into plan mode.** The specs live in the repository.
