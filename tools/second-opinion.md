# Capability: second-opinion

## Contract

An independent review of a plan or a diff by something that did not write it,
with the brief "find what the review so far missed". Returns findings only,
each with the line or plan section that motivates it. No compliments. The
result is data: it informs a decision, it does not make one.

Used through `agents/outside-voice.md` for providers 1 and 3: it runs a second
model when one is installed and, when none is present, is the reviewer itself
with fresh context. Providers 2 and 2b are Claude Code skills, which a subagent
cannot invoke; the calling command runs them itself, in the main context, when
the target is a diff, and folds their findings in next to the agent's.

Two pins, because plans and diffs differ: `Second opinion for plans` and
`Second opinion for diffs`. A project that has its own isolated review runner
for diffs typically pins `run scripts/<its script>` for diffs and `use codex`
for plans.

## Providers (first available wins; CLAUDE.md may pin one)

### 0. Project script (`run <command>`)

The command is run from the project root against the branch or diff and its
output is the review. Prefer this whenever the project has one: a project
runner can isolate the model (stripped config, no MCP, read-only tools, a
throwaway worktree) in ways provider 1 does not.

**What a `run` pin replaces.** It replaces the external-model providers
(0 and 1) only. The built-in `code-review` and `security-review` skills
(providers 2 and 2b) are local and cheap and still run, by the calling
command, when they are listed. `none` switches everything off.

### 1. Codex CLI (a different model)

**Detect:** `command -v codex`.

**Recipe** for a plan or any file:

```bash
codex exec -s read-only "Read <plan path>. The reviewers so far found: <findings>.
Find what they missed: logical gaps, unstated assumptions, a fundamentally simpler
approach, feasibility risks taken for granted, missing dependencies or sequencing
problems, and whether this is the right thing to build at all. Be terse. No compliments."
```

For a diff, a structured review against the base branch:

```bash
codex review --base <base-branch>
```

A timeout or refusal is missing coverage, not a clean bill: say so.

**Isolation note.** This runs with the user's full Codex configuration,
including any MCP servers and the sandbox policy in `~/.codex`. It sends the
plan or diff to an external service. A project that needs isolation pins a
script (provider 0) or `none`.

### 2. Claude Code built-in review, for diffs only

**Detect:** the `code-review` skill is listed. It reviews a diff, branch, PR,
or path for correctness bugs; it does not take a plan.

**Recipe:** invoke `code-review` with a level (`high` for a broad pass) and no
`--fix`; fold the findings in with the same evidence standard. Do not launch
`ultra`; it is billed and user-triggered.

### 2b. Claude Code built-in security review, for diffs only

**Detect:** the `security-review` skill is listed. A one-time pass over the
current branch's diff for common vulnerability classes.

**Recipe:** invoke `security-review` when the diff touches auth, input
handling, data access, or anything that leaves the process. Fold its findings
into the security bullet of the defect checklist. It complements provider 2,
it does not replace it.

### 3. Fresh-context Claude agent

Always available, and always runs even when provider 1 ran: it is the second
opinion on the second opinion. `agents/outside-voice.md` with only the plan path or diff
and the findings so far. Same model, different context: it catches omissions,
not blind spots the model shares with itself. Say which provider ran in the
document.

## Fallback

Provider 3 is the fallback and always runs. Record `Second opinion: fresh
context (no second model available)`.

## Project override

`CLAUDE.md`, `## Workflow`, `### Tools`: `Second opinion for plans: ...` and
`Second opinion for diffs: ...`, each `use <provider>`, `run <command>`, or
`none`. Honor `none` absolutely and record the pass as skipped by project
policy: providers 0 and 1 may send code to an external service.
