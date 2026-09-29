# Release

Cut a release of whatever this repository ships: an app, a library, a package,
a set of configs, or documentation. The shape is always the same: decide the
version, record what changed, land it on the default branch through a PR, tag
it, and let the tag or the recipe do the publishing. What differs per
repository comes from the project's `### Release` recipe, not from this file.

All git and bash commands needed for these steps are pre-approved; do not ask
for confirmation. Because they are pre-approved, arm the **destructive-guard**
capability (`~/.claude/tools/destructive-guard.md`) before step 1: the remote
branch delete and the tag push are the commands it exists for.

## What the recipe supplies

Read `CLAUDE.md`, `## Workflow`, `### Release` and `### CI policy`, and
`.claude/workflow/release.md` if it exists. The recipe answers these; the
defaults apply where it is silent:

| Question | Default when the recipe is silent |
| --- | --- |
| Where does the version live? | The first that exists: `VERSION`, `pyproject.toml`, `package.json`, `Cargo.toml`; else tags only |
| Version scheme? | Whatever the last tag used; if no tags, ask |
| Which files must carry the version? | The one file found above |
| Is there a changelog to promote? | `CHANGELOG.md` with `## [Unreleased]`, if present |
| Branch and PR, or direct to default branch? | Branch `release/v{VERSION}`, PR, merge, tag on the default branch |
| Tag format and extra tags? | `v{VERSION}`, no extras (`ask` in the recipe means ask per release) |
| Does the pre-PR gate apply to the release PR? | No: a diff of only version files and the changelog was reviewed in its feature PRs |
| What publishes the release? | Nothing beyond the tag; CI is assumed to react to it |
| Runbook for anything else? | None |

If the repository ships nothing that is versioned (a config repo, a docs repo
with no tags), the release is the PR itself: skip the version and tag steps,
say so, and stop after the merge.

## Process

Before step 1, read the recipe as described above; its preconditions are
instructions. If neither the contract nor an addendum exists, proceed on the
defaults and mention once that `bash ~/.claude/bin/init-project` sets the
project up.

1. **Verify readiness**:
   - Run `git status` and `git diff --stat`; note what is uncommitted
   - Run the contract's `### Checks` full tier and confirm it passes
   - If the changelog has no entries covering this release, run `/changelog`
     first and verify again
   - If `### CI policy` names a local pre-PR gate (a review or audit script)
     and the recipe says it applies to release PRs, it must have run clean on
     this branch; run it now if not. By default a version-only release PR is
     exempt

2. **Decide the version**:
   - Read the current version from where the recipe says it lives
   - Derive the next one from the scheme: calendar versions take today's date
     (append `.N` for a same-day release), semantic versions bump per the
     changelog's categories (breaking → major, added → minor, else patch)
   - State the version and the reason in one line before changing anything

3. **Bump and promote**:
   - Write the version into every file the recipe lists, in the format each
     requires; all must agree with the tag
   - Promote `## [Unreleased]` in the changelog to `## [v{VERSION}] - {date}`
     and add a new empty `## [Unreleased]` above it, unless the recipe says the
     changelog is handled differently
   - Do anything else the recipe's runbook requires before the commit

4. **Branch and commit**:
   - If on the default branch, create `release/v{VERSION}`; otherwise stay on
     the current branch
   - Stage the changed files by name (never `-A` or `.`)
   - Commit with message `Release v{VERSION}`

5. **Push and open the PR**:
   - Push with `-u`
   - `gh pr create` with title `Release v{VERSION}` and the changelog section
     for this version as the body
   - Print the PR URL. Opening the PR is a CI trigger; say so

6. **Watch checks and reviews**:
   - Poll `gh pr checks` until every check passes or one fails
   - On a failure, report it and STOP
   - Address reviewer comments, commit, push
   - **After a follow-up push, consult `### CI policy`.** If pushes re-run CI,
     resume polling. If they do not, or only the user may re-trigger, report
     that the branch is pushed and CI has not been re-triggered, and STOP.
     Never re-trigger CI on your own initiative

7. **Wait for merge**:
   - Tell the user the PR is ready to merge
   - Poll `gh pr view --json state` every 30-60 seconds for up to ten
     minutes. If it is not merged by then, STOP and tell the user to run
     `/release` again once it is; on re-entry, detect the merged PR from the
     branch name and continue at step 8

8. **Tag and clean up**:
   - Check out the default branch and pull
   - Delete the release branch locally and remotely
   - Tag the merge commit `v{VERSION}`. For extra tags: add the ones the
     recipe defines; when it says `ask`, ask now; when it says nothing, add none
   - Push each tag by name (`git push origin v{VERSION}`), never `--tags`,
     which would push every stray local tag. If the recipe names a publish step that the tag does not
     trigger (a package upload, a docs deploy), run it now, exactly as written;
     never invent one
   - Confirm completion with the tags and, when there is one, the published
     artifact's location

## Guidelines

- **The recipe wins.** Version files, scheme, tags, and publish steps come from
  the project; this command supplies the sequence
- **Everything the tag must match, matches.** Refuse to tag while a version
  file disagrees
- **Never tag or publish an unmerged branch**, and never push straight to the
  default branch
- **Never re-trigger CI** on your own initiative; the CI policy decides who does
- **Say what you skipped.** A repo with nothing versioned still gets a clear
  "no version or tag for this repository, the PR is the release"
