# Plan Feature

Turn an idea for a new feature into a feature document that describes the desired
outcome for the users, the user experience, how success will be measured, and a
sketch of our hypothesis for how it should work. This is the first step for any
new feature and it happens **before** looking at the codebase.

Why this step exists: starting a feature in `/research_codebase` anchors the
thinking on the code that already exists. This command anchors it on the users
and the outcome instead. The codebase gets its turn in the next step.

## Arguments

$ARGUMENTS - Optional: a description of the feature idea, or a path to a file with notes

## Where documents live

The layout of `thoughts/` is defined by the project's `thoughts/README.md`,
which is authoritative; the default it starts from is
`~/.claude/templates/thoughts-README.md`: `features/`, `research/`, `plans/`,
`verifications/` (dated snapshots) and `reference/`, `todo/` (living). **A
directory is a *kind* of document, never a *status*.** If this command needs a
directory the project's README does not list, ask before creating it, and add
the row to the README when the user agrees; when running autonomously, stop and
report instead. Never move a document because its status changed, and never
create a `completed/` directory.

Feature documents go in `thoughts/features/YYYY-MM-DD-slug.md`. The **date is
the status**: a feature document is a snapshot of what we wanted and why, true
as of the day it was written. It is not edited afterwards except to correct an
error. The hypothesis gets refined by research and turned into decisions by the
plan; those refinements live in *their* documents, not back here. If the desired
outcome itself changes, write a new feature document.

**This command mints the slug.** Research, plan, and verification all reuse it:
`thoughts/features/2026-09-17-offline-sync.md` →
`thoughts/research/2026-09-19-offline-sync.md` →
`thoughts/plans/2026-09-20-offline-sync.md` →
`thoughts/verifications/2026-09-28-offline-sync.md`. Pick a slug that names the
outcome, not the implementation (`offline-sync`, not `sqlite-queue`).

Feature documents carry **no `status:` frontmatter** — only plans do.

## Process

### 0. Project conventions

Before anything else, read the project's `CLAUDE.md` `## Workflow` section and,
if it exists, `.claude/workflow/plan_feature.md`. They override the defaults in this
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

### 1. Capture the idea

- If a description or file path was provided as argument, read it fully and use it as the starting point
- If nothing was provided, ask: "What feature do you have in mind? Describe it in a few sentences — who it is for and what it should make possible."
- Read `CLAUDE.md` and `README.md` (or equivalent product docs) **only** to pick up product vocabulary, the names of user roles, and existing success metrics. Do not open source files.
- If `thoughts/features/` already has a document on the same topic, read it and ask whether this supersedes it or is a different feature

### 2. Work through the feature with the user, one topic at a time

Do this as a conversation, in rounds. Ask, listen, restate, and move on only when
the answer is concrete. Do not present the whole template and ask the user to
fill it in.

1. **Problem and users** — Who has the problem? What do they do today, and what does it cost them? If there are several user groups, name each and say which one this feature is primarily for. Push for three things before moving on:
   - **A named human, not a category.** "Field technicians" is a filter, not a person. Ask for a role, what that person is measured on, and what this problem costs them specifically.
   - **Demand evidence.** What is the strongest sign someone actually wants this? Not "interested" or "signed up", but who would be upset if it disappeared, or what they do today to get around its absence. If the honest answer is "we think so", record that as a hypothesis, not a fact.
   - **The status quo.** "Nothing, there is no solution today" is a red flag, not an opportunity. If nobody has cobbled together a workaround, the problem may not be painful enough. Ask again.
2. **Premise check** — Before going further, state two premises and ask the user to agree or disagree with each:
   - *This is the right problem.* A different framing would not give a dramatically simpler or more impactful solution.
   - *Doing nothing is not acceptable.* Name what it costs to leave this as it is.
   If the user disagrees with either, go back to topic 1 with the new framing. Record the agreed premises; they go in the document.
3. **Desired outcome** — When this feature exists, what is different for those users? State it as an observable change in their world, not as a capability of the software ("a driver can finish the day without re-entering any stop", not "add offline support"). Then bracket the ambition in two questions:
   - **The 10x version.** What is the version of this that would make the user think "they really thought of everything"? Describe it concretely, from the user's side.
   - **The smallest version.** What is the smallest thing that still delivers the outcome? Not a demo, the real outcome.
   The gap between the two answers is the raw material for Not in scope (topic 7). Decide with the user where this round lands.
4. **User experience** — Walk through the experience as a short narrative from the user's point of view: what they see, what they do, what happens, where it ends. Cover the main path and the one or two most likely awkward paths (nothing to show, action fails, user changes their mind).
5. **Success measures** — What would we look at, after shipping, to know whether it worked? Prefer measures that already exist or can be observed cheaply. For each measure, note how it would actually be observed (a log line, a metric, a support ticket count, a manual check). If the honest answer is "we would not know", write that down — it is a research question.
6. **Hypothesis** — Sketch two or three ways it could work, at the level of moving parts and where the data flows, not classes and files. One must be the minimal version (fewest moving parts), one the ideal version (what we would build with no constraints), and optionally one lateral version (a different framing of the problem). Give a one-line recommendation and ask the user to pick. The chosen one becomes the Hypothesis; the rejected ones are recorded in one line each with the reason. Label all of it as untested: it is what research will confirm, correct, or replace.
7. **Not in scope** — What adjacent things are we deliberately not doing in this round? Start from the gap between the 10x and smallest versions in topic 3. Every item gets a one-line reason; a deferral without a reason is a guess.
8. **Open questions** — What do we need to find out before we can plan this? Separate what needs the codebase from what needs the outside world (users, the web, a vendor). These become the research questions for `/research_codebase`.

Push back gently when an answer is an implementation ("use a background job") where an outcome or experience is wanted. Record the implementation idea under Hypothesis and ask again for the outcome.

### 3. Write the feature document

Save to `thoughts/features/YYYY-MM-DD-slug.md`. If the project's
`thoughts/README.md` does not list `features/`, ask before creating it (see
"Where documents live"):

```markdown
# Feature: [Name]

**Date:** YYYY-MM-DD
**Author:** [who asked for this / who wrote it]

## Problem

[Who has the problem, what they do today, what it costs them. 1-2 paragraphs.]

**Demand evidence:** [the strongest sign someone wants this: a behaviour, a workaround, a cost. Or: "none yet, hypothesis".]

## Users

- **[Primary user group]** — [what they need from this feature. Name a concrete person or role and what they are measured on.]
- **[Secondary user group]** — [how they are affected, if at all]

## Premises

1. [This is the right problem because ...] — agreed YYYY-MM-DD
2. [Doing nothing costs ...] — agreed YYYY-MM-DD

## Desired Outcome

[What is different for the users when this exists. Observable, not a capability list. 2-4 sentences.]

## User Experience

**Main path:**
1. [User does / sees ...]
2. [...]
3. [Ends with ...]

**Edge paths:**
- **[Situation]** — [what the user experiences]
- **[Situation]** — [what the user experiences]

## Success Measures

| Measure | Target | How we observe it |
| --- | --- | --- |
| [e.g. stops re-entered per day] | [e.g. drops to zero] | [e.g. existing `stop.created` audit log] |
| [...] | [...] | [...] |

[Note anything we cannot currently observe — that is a research question below.]

## Hypothesis (untested)

[How we think it should work: the moving parts, where data lives, what triggers what. A short paragraph or a small list. This is a sketch for research to test, not a design.]

**Alternatives considered:**
- **[Minimal / Ideal / Lateral: name]** — [one line on what it was and why it was not chosen]
- **[...]** — [...]

## Not In Scope

- [Explicit out-of-scope item] — [one-line reason]
- [Another] — [one-line reason]

## Open Questions

### About the codebase
- [Question research must answer by looking at the code, e.g. "Is there an existing queue we can reuse?"]
- [Question about whether a success measure can be observed today]

### About users and the outside world
- [Question that needs a user, the web, a vendor, or a standard]
```

### 4. Present and hand off

**Joining an existing chain.** If research or a plan for this outcome already
exists under another slug, do not mint a new one: reuse that slug, add a
`**Feature:**` line to the existing research document if it lacks one, and
point "Next" at the step that is actually missing rather than at
`/research_codebase`.


Show the user the document path and a six-line summary: the outcome, the
primary user, the headline success measure, the hypothesis in one sentence, the
number of open questions, and anything the user was asked and declined to
decide ("Undecided: none" when there is nothing). Then point to the next step:

```
Next: /research_codebase thoughts/features/YYYY-MM-DD-slug.md
```

## Guidelines

- **Do not explore the codebase** — no Explore or codebase-analyzer agents, no reading source files. The whole reason this command exists is to think about the outcome before the code. Note codebase questions under Open Questions and let `/research_codebase` answer them.
- **Do not switch into plan mode** - we want all documents in the repository
- **Do not make any code changes**
- **Outcome over capability** — every section should be readable by someone who has never seen the code. If a sentence only makes sense to a developer, rewrite it from the user's side.
- **Concrete user experience** — a narrative someone could act out beats a list of requirements
- **Every success measure says how it is observed** — a measure nobody can observe is an open question, not a measure
- **Label the hypothesis as a hypothesis** — research and planning are allowed to replace it, and the document should make that easy
- **Open Questions is the contract with `/research_codebase`** — write questions that can be answered, not themes
- **One hypothesis is not enough** — the user picks from alternatives, and the rejected ones stay in the document in one line each; knowing what we chose not to do is worth as much as what we chose
- **Push once, then push again** — the first answer to a hard question is usually the polished version; the real one comes after the second push. Move on when the answer is concrete, not when the user stops talking
- **Record what the user declined to decide** — never silently pick a default for them
- **Mint the slug here** — it names the outcome and is reused by research, plan, and verification
