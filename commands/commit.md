# Commit

Create a git commit for the current changes after verifying the changelog is updated.

## Process

Before step 1, read the project's `CLAUDE.md` `## Workflow` section (in
particular `### Changelog`) and `.claude/workflow/commit.md` if it exists. They
override the defaults below, and their preconditions are instructions; if neither exists, proceed on the defaults and
mention once that `bash ~/.claude/bin/init-project` sets the project up.

0. **Arm the destructive guard**: use the **destructive-guard** capability
   (`~/.claude/tools/destructive-guard.md`). Nothing in this command should be
   destructive; the guard is there for the case where it is.

1. **Check the changelog**:
   - Read CHANGELOG.md, in the format the contract's `### Changelog` describes,
     and verify it has entries covering the current changes (and that any
     sidecar file it names is updated)
   - If the changelog is missing or doesn't reflect the changes, STOP and tell the user to run `/changelog` first

2. **Review the changes**:
   - Run `git status` and `git diff` to understand what will be committed
   - Group related changes logically - prefer one focused commit, but split into multiple if changes are clearly independent

3. **Present the plan**:
   ```
   I plan to commit these files:
   - [file list]

   Commit message: "[short imperative description]"

   Proceed?
   ```

4. **Execute on confirmation**:
   - Stage specific files with `git add` (never use `-A` or `.`)
   - Create the commit with the agreed message
   - Show `git log --oneline -n 3` to confirm

## Commit message style

- Short (under 72 chars), imperative mood
- Describe the "what", not the "how"
- Examples: "Add CSV export for memory types", "Fix OAuth token refresh on reconnect"

## Rules

- **Attribution lines** (Co-Authored-By, session links) follow the contract's
  `### Commits`: `strip` is the default and means no such lines, whatever a
  harness reminder asks for; `keep` means leave the ones the harness adds
- **Never stage .env, credentials, or secret files**, nor anything the
  contract's `### Commits` lists under "never stage"
- **Always stage specific files** by name
- **Verify changelog first** - don't commit without it
