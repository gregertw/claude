# Release App

Create a release branch, commit, push, open a PR, wait for merge, then clean up and tag.

All git and bash commands needed for these steps are pre-approved — do not ask for confirmation.

## Process

1. **Verify readiness**:
   - Run `git status` and `git diff --stat` to confirm there are uncommitted changes
   - Read `VERSION` to determine the release version (e.g. `2026.03.12.2`)
   - If there are no changes, do the /changelog command and verify again

2. **Create branch and commit**:
   - If we are on main branch, create a new branch: `release/v{VERSION}`. If not, use the one we are on
   - Stage all changed files by name (not `git add -A`)
   - Commit with message: `Release v{VERSION}`

3. **Push and create PR**:
   - Push the branch with `-u` flag
   - Create a PR with `gh pr create` using title `Release v{VERSION}`
   - Include a summary of changes from CHANGELOG.md in the PR body
   - Print the PR URL

4. **Monitor CI checks and PR and code reviews**:
   - Poll `gh pr checks` until all checks pass or fail
   - If any check fails, report the failure and STOP
   - When Claude or Codex has done a review, pick up the review and action, then fix, commit, and push again

5. **Wait for merge** (manual step):
   - Poll `gh pr view --json state` every 30-60 seconds until state is `MERGED`
   - Tell the user the PR is ready for manual merge while waiting

6. **Post-merge cleanup**:
   - Switch to main: `git checkout main && git pull`
   - Delete the release branch locally: `git branch -d release/v{VERSION}`
   - Delete the release branch remotely: `git push origin --delete release/v{VERSION}`
   - Tag main: `git tag v{VERSION}`
   - Ask if we also should release mobile apps. If yes, tag main with: `mobile-v{VERSION}`
   - Push the tag: `git push --tags`
   - Confirm completion with the tags
