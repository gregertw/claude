## Workflow

This repository uses the workflow and slash commands from
<https://github.com/gregertw/claude> (installed at `~/.claude`; the process is
described in that README, installed as `~/.claude/WORKFLOW-README.md`). The commands read the subsections below. Keep each
one short and literal; the commands treat them as instructions, not prose.
Delete a subsection you do not need. A per-command addendum can be added in
`.claude/workflow/<command>.md` (see `.claude/workflow/README.md`).

### Checks

Two tiers. **Fast** runs after every implementation phase and after each group
of iteration changes. **Full** runs before every commit, during verification,
and before a release. All must pass. If the repo has no meaningful split, put
everything under Full and leave Fast empty: the commands then run Full
everywhere.

Fast:

```bash
# example; replace with this repo's commands, or leave empty
poetry run ruff check .
poetry run pyright
poetry run pytest tests/unit
```

Full:

```bash
# example for a python + node repo; replace with this repo's commands
poetry run ruff check .
poetry run pyright
poetry run pytest
npm --prefix frontend run lint
npm --prefix frontend run test:run
npm --prefix frontend run build
```

Preconditions and warnings (the commands follow these as instructions):
[e.g. "integration tests need the dev server on :5000 started with `...`";
"do not edit files while the suite runs, the reloader wedges it";
"after switching branches run `poetry install` first"; or "none"]

### App

- Start for checks: `[command; may differ from the QA start, e.g. no tunnel]` (`http://localhost:PORT`)
- Start for browser QA: `[command]` (base URL `[http://localhost:PORT or the tunnel host]`)
- Allowed hosts for browser QA: `localhost`, `127.0.0.1` [add a preview or tunnel host when login only works through it; it still serves the local app]
- Seed or reset data: `[command, or "none"]`
- Stop: `[command, or "none"]`

### Test account

[Either `none`, or a throwaway account the browser agent MAY type into a login
form, stated explicitly:]

- Account: `[username]` / password `[value]`, throwaway, local or preview only
- The browser agent may type these credentials: yes
- Seed with: `[command]`

### Commits

- Attribution lines (Co-Authored-By, session links added by the harness): strip [default] / keep
- Never stage: `.env*` [add generated bundles, build output]

### CI policy

- Opening a PR: [triggers CI / does not]
- Pushing to an open PR: [re-runs CI / runs nothing]
- Re-triggering CI: [only the user, never the agent / the agent may, via `<command>`]
- Local pre-PR gate: `[command, e.g. scripts/review-local.sh, or "none"]`

### Tools

No pins: every capability auto-detects (run `bash ~/.claude/bin/detect-tools`
to see what). Add a line only to override. Accepted forms: `use <provider>`,
`run <command>`, `none`; the provider names are tabled in
`~/.claude/tools/README.md`. Examples (indented so they are not read as pins;
copy one to the left margin to activate it):

    - Browser QA: use claude-in-chrome
    - Second opinion for plans: use codex
    - Second opinion for diffs: run scripts/review-local.sh
    - Diagrams: use mermaid-fence
    - Web search: none

### Changelog

- File: `CHANGELOG.md`, format: [Keep a Changelog under `## [Unreleased]` / other]
- Entry style: [one line imperative / multi-line user-facing prose; no file paths]
- Sidecar files to keep in sync: [`frontend/src/data/whatsNew.ts` (user-visible subset) / none]
- Categories: [Added, Changed, Fixed, Removed / project's own]

### Release

- Version files: [`VERSION` as `YYYY.MM.DD`; `pyproject.toml` and `frontend/package.json` as PEP 440 / single file / none]
- Changelog promotion: [promote `## [Unreleased]` to `## [vX] - date` and add a new empty Unreleased / none]
- Branch and PR: [release/vX branch, PR to main, merge, tag on main / direct]
- Tags: [`vX`; extra tags: `none` / `ask` (the command asks per release) / `mobile-vX` always]. Tags are pushed by name.
- Pre-PR gate for release PRs: [skip (default: a version-only diff was reviewed in its feature PRs) / run]
- Runbook: [`docs/RELEASE.md` / none]

### Dependencies

- [`../sibling-repo` is an editable path dependency: bisect it in a worktree, never move its HEAD / none]

### Thoughts

- Layout: `thoughts/README.md` is authoritative. Default: `features/`, `research/`, `plans/`, `verifications/`, `reference/`, `todo/`.
- [Additional directories or rules the commands must follow when they write here, e.g. "`todo/` has an `INDEX.md`: add a row when adding a todo, remove it with the file", or "`inbound/` holds untriaged external reports"]
