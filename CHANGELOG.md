# Changelog

All notable changes to this workflow are recorded here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Releases use
calendar versions (`vYYYY.MM.DD`, with `.N` appended for a second release the
same day). Changes from before the first release are grouped by the date they
landed.

## [Unreleased]

## [v2026.11.17] - 2026-11-17

### Added

- `/create_spec` builds living specs for an existing repository: with no
  argument it proposes the capability map and writes the index; with a
  capability it drafts that spec from tests, docs, plans and code, and reviews
  every discrepancy and code-only behaviour with you so today's bugs do not
  become requirements.
- Living specs in `thoughts/specs/` describe how each capability behaves
  today, with stable statement IDs, source tags, and `draft` / `reviewed` /
  `partial` status. Plans carry a Spec Delta, and `/verify_implementation`
  merges it into the spec after a clean whole-plan verification, recording
  `spec_merged:` on the plan.
- Acceptance criteria: numbered, stable pass/fail statements about observable
  behaviour. `/plan_feature` writes them from the user experience (or
  `/create_plan` when there is no feature document), and every plan maps each
  one to a phase and a named test in a Traceability table.
- Plan approval gate: an `approved:` date in the plan's frontmatter, written
  only on the user's explicit approval. `/implement_plan` does not start
  without it, and a substantive `/update_plan` change removes it.
- `/implement_plan` writes the acceptance tests first and confirms each one
  fails on the missing behaviour before writing the code.
- Small-change lane in `/create_plan`: a one-phase change of about five files,
  with no data model, auth or integration work, can be planned straight from a
  description without a research document or the evaluator team.
- `/verify_implementation` checks that each acceptance criterion has a test
  that asserts its result; an uncovered criterion is an issue of the same
  severity as the behaviour being broken.
- `/research_codebase` reports how each acceptance criterion could be tested
  and what the current spec says about the capabilities the topic touches.
- `/fix_bug` adds the missing acceptance criterion and spec statement when a
  bug shows the expected behaviour was never written down.
- `shared/conventions.md`: one set of rules every command reads at its first
  step (project conventions, document layout, plan frontmatter, acceptance
  criteria, living specs, iterations log). `bin/install` now copies `shared/`.
- The `### Thoughts` section of the `CLAUDE.md` contract names where the
  project's user-facing docs live and where its specs live.
- This changelog.

### Changed

- Post-implementation refinements go to a sibling iterations log
  (`thoughts/plans/YYYY-MM-DD-slug.iterations.md`) in one-to-three-line
  entries, so the plan stays a readable design. Plans with an older in-plan
  Iterations section keep it and point to the log.
- Commands refer to `shared/conventions.md` instead of each repeating the
  layout, frontmatter and project-convention rules.
- Plan evaluation notes are one table of findings with their disposition.
- Plans describe behaviour, files and interfaces rather than code, and a plan
  over roughly 600 lines is checked for a phase that should be its own plan.
- A `proposed` plan now means "not started" rather than "not agreed"; approval
  is recorded separately.
- `bin/init-project` creates `thoughts/specs/`, and the default
  `thoughts/README.md` lists it.

## 2026-11-01

### Added

- `/research_codebase` sweeps the project's documentation for each moving
  part: which pages state the behaviour, which would become false, and which
  are already wrong. The research document gains a Documentation Impact
  section.

## 2026-09-29 — Prompt audit (PR #1)

### Changed

- The outside-voice reviewer always adds its own fresh-context pass, and its
  report lists every provider that ran.
- Active plans are routed to `/implement_plan` rather than between
  `/update_plan` and `/iterate_plan`.
- The browser credential rule matches the contract's `### Test account`.
- Commands allow decisions the user explicitly defers.
- Web research happens where relevant and honours a `none` tool pin; the
  query-sanitising rule is stated in the research agent itself.
- The contract's check tier is run verbatim instead of hardcoded examples.
- The recommendation guideline matches the plan template.
- Scope rules are one reasoned section instead of repeated CRITICAL / DO NOT
  warnings; the documentarian agent no longer judges defects.

### Removed

- "Ultrathink" and "think deeply" depth prose, replaced by a citation rule.
- Numeric search budgets.
- A browser-search fallback the agent could not use.

## 2026-09-17

### Added

- `/plan_feature`: a feature definition, one topic at a time, before anyone
  reads the code (problem, premise check, outcome, 10x and smallest versions,
  user experience, success measures, hypotheses, scope, open questions).
- `/fix_bug`: root cause before any change, a regression test that fails
  before and passes after, and a dated investigation record.
- `/release`: a generic, recipe-driven release for apps, libraries, packages
  and unversioned repos.
- `browser-qa` and `outside-voice` adapter agents.
- Capability layer in `tools/` (browser, second opinion, scope lock,
  destructive guard, diagram, web search) with providers, detection, fallback
  and project pins.
- Project templates (the `## Workflow` contract for `CLAUDE.md`, the
  `thoughts/README.md`, the addenda README) and the `bin/install`,
  `bin/init-project` and `bin/detect-tools` scripts.
- Every command reads the project contract and a per-command addendum before
  its first step, runs checks in fast and full tiers, and does not block when
  run autonomously.
- `/create_plan` evaluates the draft from five angles plus an outside voice;
  `/verify_implementation` hunts defects with a named checklist and an
  adversarial pass, and can be re-run in a loop with `/iterate_plan`.
- README rewritten: install steps, a guide for an AI installing the workflow,
  the process, customizing, agents and scripts.

### Removed

- Invocations of external skill suites from the commands; their useful angles
  are folded into the workflow's own steps.
- `/release_app`, replaced by `/release`.

## 2026-07-27

### Added

- Document hierarchy under `thoughts/`: a directory is a kind of document,
  dated files are snapshots, undated files are living, and one slug ties a
  piece of work together.
- Plan status in frontmatter from a closed vocabulary (`proposed`, `active`,
  `done`, `superseded`), with links to the verification or replacement.

## 2026-06-17

### Added

- `/release_app` command.

## 2026-03-04

### Changed

- `/create_plan` resolves decisions before designing the approach;
  `/implement_plan` treats unrelated failing tests as failures to investigate;
  `/research_codebase` identifies decisions rather than recommending.

## 2026-02-27

### Added

- `/iterate_plan` for refinements after implementation.
- The work process described in the README.

## 2026-02-12

### Added

- `/update_plan` for changes to a plan before work starts.
- `/create_plan` asks for additional context and evaluates the plan from
  architecture, security, scalability and usability angles.

## 2026-02-11

### Added

- Initial commands: `/research_codebase`, `/create_plan`, `/implement_plan`,
  `/verify_implementation`, `/changelog`, `/commit`.
- `codebase-analyzer` and `web-search-researcher` agents.
- README and CC BY 4.0 license.
