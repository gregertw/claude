# Capability: destructive-guard

## Contract

Warn before, or block, destructive shell commands during a session where git
and shell commands are pre-approved: recursive deletes, force-push, reset
--hard, dropping tables, deleting remote branches. The guard is a tripwire, not
a policy boundary.

## Providers (first available wins; CLAUDE.md may pin one)

### 1. gstack careful

**Detect:** the `careful` skill is listed, and
`$HOME/.claude/skills/gstack/careful/bin/check-careful.sh` exists.

**Recipe:** invoke `/careful` at the start of the command. A PreToolUse hook on
Bash then warns on destructive patterns and hard-blocks a force-push to the
default branch and recursive deletes of `/` or the home directory.

### 2. Self-check

Always available. Before any command matching `rm -r`, `push --force`,
`push -f`, `reset --hard`, `branch -D`, `push origin --delete`, `DROP`,
`TRUNCATE`, or `kubectl delete`, print the command and what it destroys, and
confirm it is the intended target. Prefer `--force-with-lease` to `--force`.
Never force-push the default branch.

## Fallback

Pinned `none`: skip and record `Destructive guard: skipped by project policy`.

Provider 2.

## Project override

`CLAUDE.md`, `## Workflow`, `### Tools`: `Destructive guard: use <provider>`.
