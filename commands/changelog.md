# Changelog

Update CHANGELOG.md based on all changes in the current branch, including uncommitted changes.

## Process

1. **Identify all changes** in this branch vs main:
   ```
   git diff main...HEAD
   git diff
   git diff --cached
   ```

2. **Read the existing CHANGELOG.md** to understand the current format and avoid duplicates

3. **Categorize each change** using Keep a Changelog categories:
   - **Breaking** - incompatible API or behavior changes
   - **Added** - new features
   - **Changed** - changes to existing functionality
   - **Fixed** - bug fixes
   - **Removed** - removed features

4. **Write one line per change** under the `## [Unreleased]` heading. Each line should be concise and describe the user-visible impact, not implementation details. Skip categories with no entries.

5. **Do not duplicate** entries already in the changelog

## Format

Follow the existing format in CHANGELOG.md:
```markdown
## [Unreleased]

### Added
- Short description of what was added

### Fixed
- Short description of what was fixed
```

## Rules

- One line per item - no sub-bullets or multi-line descriptions
- Focus on what changed from the user's perspective
- Use imperative mood ("Add X" not "Added X" or "Adds X")
- Don't include internal refactoring unless it affects behavior
- Review the full diff to catch everything, don't rely on commit messages alone
