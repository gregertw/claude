# Commit

Create a git commit for the current changes after verifying the changelog is updated.

## Process

1. **Check the changelog**:
   - Read CHANGELOG.md and verify it has entries covering the current changes
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

- **No co-author lines or Claude attribution** in commits
- **Never stage .env, credentials, or secret files**
- **Always stage specific files** by name
- **Verify changelog first** - don't commit without it
