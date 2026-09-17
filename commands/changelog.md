# Changelog

Update CHANGELOG.md based on all changes in the current branch, including uncommitted changes.

## Process

Before step 1, read the project's `CLAUDE.md` `## Workflow` section (in
particular `### Changelog`) and `.claude/workflow/changelog.md` if it exists. They
override the defaults below, and their preconditions are instructions; if neither exists, proceed on the defaults and
mention once that `bash ~/.claude/bin/init-project` sets the project up.

1. **Identify all changes** in this branch vs the remote default branch (a
   stale local `main` pulls already-merged commits into the diff):
   ```
   git fetch origin main
   git diff origin/main...HEAD
   git diff
   git diff --cached
   ```

2. **Read the existing CHANGELOG.md** and the contract's `### Changelog`
   subsection. The project's observed format and stated conventions win over
   the defaults under "Format" and "Rules" below: entry length, mood,
   categories, and what is banned (file paths, internal names). If the
   contract names sidecar files (a user-visible "what's new" list, release
   notes), update them in the same pass with the entries that belong there

3. **Categorize each change** using Keep a Changelog categories:
   - **Breaking** - incompatible API or behavior changes
   - **Added** - new features
   - **Changed** - changes to existing functionality
   - **Fixed** - bug fixes
   - **Removed** - removed features

4. **Write one line per change** under the `## [Unreleased]` heading. Each line should be concise and describe the user-visible impact, not implementation details. Skip categories with no entries.

5. **Do not duplicate** entries already in the changelog

## Format (default, yields to the project's)

Follow the existing format in CHANGELOG.md:
```markdown
## [Unreleased]

### Added
- Short description of what was added

### Fixed
- Short description of what was fixed
```

## Rules (defaults, yield to the project's `### Changelog`)

- One line per item - no sub-bullets or multi-line descriptions, unless the
  project writes multi-line prose, in which case match it
- Focus on what changed from the user's perspective
- Use imperative mood ("Add X" not "Added X" or "Adds X") unless the existing
  entries use another mood consistently
- Don't include internal refactoring unless it affects behavior
- Review the full diff to catch everything, don't rely on commit messages alone
