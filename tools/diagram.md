# Capability: diagram

## Contract

Turn a described flow, state machine, or pipeline into a diagram that lives
next to the document that needs it and stays editable. Mermaid source is the
interchange format: it is text, it diffs, and GitHub renders it inline.

## Providers (first available wins; CLAUDE.md may pin one)

### 1. gstack diagram

**Detect:** the `diagram` skill is listed.

**Recipe:** write the mermaid source first, then invoke `/diagram <path to
.mmd>`. It produces an editable `.excalidraw` scene plus rendered SVG and PNG,
fully offline. Keep the outputs next to the document, for example
`thoughts/reference/diagrams/<slug>.{mmd,excalidraw,svg,png}`, and link the
SVG from the document.

### 2. Mermaid fence

Always available. Put the mermaid source in a fenced block in the document:

    ```mermaid
    flowchart LR
      A[Request] --> B{Cached?}
      B -- yes --> C[Serve]
      B -- no --> D[Fetch] --> C
    ```

GitHub and most markdown viewers render it. No files, no tools.

## Fallback

Pinned `none`: skip and record `Diagrams: skipped by project policy`.

Provider 2. For a reader without a mermaid renderer, add a one-paragraph prose
description under the fence.

## Project override

`CLAUDE.md`, `## Workflow`, `### Tools`: `Diagrams: use <provider>`.
