# Capability: scope-lock

## Contract

Restrict file edits to a named set of paths for the rest of a task, so a
minimal-diff rule is enforced rather than advisory. Lock after the root cause
is known, unlock when the fix is proven. A fix normally spans the code, its
tests, and the changelog; the lock is that set, not one directory.

## Providers (first available wins; CLAUDE.md may pin one)

### 1. gstack freeze

**Detect:** the `freeze` skill is listed, and
`$HOME/.claude/skills/gstack/freeze/bin/check-freeze.sh` exists.

**Recipe:** invoke `/freeze <directory>`. A PreToolUse hook then denies Edit
and Write outside it. Invoke `/unfreeze` when the fix is proven. Say in the
document that the lock was on. **It locks one directory**, so it fits only when
code, tests, and changelog all sit under one path; when a fix spans several
(the usual case), pin `Scope lock: use self-check` instead.

### 2. Self-check

Always available, and takes a set. State the boundary explicitly ("Edits
restricted to `src/billing/`, `tests/billing/`, `CHANGELOG.md` for this fix")
and, before every edit, check the path against it. If a fix genuinely needs a file outside the boundary, stop and say why
before widening it.

## Fallback

Pinned `none`: skip and record `Scope lock: skipped by project policy`.

Provider 2. Record `Scope lock: self-check` in the document.

## Project override

`CLAUDE.md`, `## Workflow`, `### Tools`: `Scope lock: use <provider>`.
