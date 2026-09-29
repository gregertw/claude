# thoughts/ — what goes where

The thinking behind a piece of work lives here, next to the code, as markdown
written by the workflow commands (see <https://github.com/gregertw/claude>). A reviewer reads
the thought process, not only the diff.

## The directories

A directory is a *kind* of document, never a *status*. Nothing moves when it
is finished.

| Directory | Holds | Dated? | Written by |
| --- | --- | --- | --- |
| `features/` | What we want to achieve — outcome, user experience, success measures, hypothesis | yes | `/plan_feature` |
| `research/` | What we found out — investigation, measurement, analysis, bug investigations | yes | `/research_codebase`, `/fix_bug` |
| `plans/` | What we intend to do — phased implementation plans | yes | `/create_plan` |
| `verifications/` | Evidence a plan actually landed | yes | `/verify_implementation` |
| `reference/` | Durable internal knowledge — protocol flows, runbooks, indexes | no | by hand |
| `todo/` | Known work not yet scheduled | no | by hand |

Add a row here before adding a directory. The commands read this table and ask
before creating a directory it does not list.

## Dated vs living

- **Dated** (`YYYY-MM-DD-slug.md`) is a snapshot: true as of that date, not
  edited afterwards except to correct an error.
- **Undated** (`slug.md`) is living: edited in place, deleted when it stops
  being true.
- If you are editing a dated file to keep it accurate, it is in the wrong
  directory.

## Same slug = same thread of work

`/plan_feature` mints the slug; research, plan, and verification reuse it.
Pick a slug that names the outcome, not the implementation.

```
features/2026-09-17-offline-sync.md
research/2026-09-19-offline-sync.md
plans/2026-09-20-offline-sync.md
verifications/2026-09-28-offline-sync.md
```

## Status lives in the plan, not in the path

Only plans carry a `status:` in their frontmatter, from a closed vocabulary:
`proposed` (written, not agreed), `active` (being implemented now), `done`
(implemented; link the verification), `superseded` (overtaken; link the
replacement). The commands write these themselves. Never create a
`completed/` directory; a finished plan stays in `plans/` because the
verification links to it by path.

## todo/ holds only what is not done

When work lands, the todo file is deleted, not annotated. The record of
finished work is the plan and the verification. A todo that grows into phases
becomes a plan via `/create_plan`.
