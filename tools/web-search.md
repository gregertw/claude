# Capability: web-search

## Contract

Find current information outside the codebase: documentation, known issues,
conventions, vendor behaviour. Returns sources with quotes and dates. Search
queries never contain hostnames, file paths, secrets, customer data, or raw
error text with any of those in it.

Used through `agents/web-search-researcher.md`.

## Providers (first available wins; CLAUDE.md may pin one)

### 1. Claude Code WebSearch and WebFetch

**Detect:** the `WebSearch` tool is listed. This is the default and the agent
already uses it.

### 2. Browser capability

When WebSearch is unavailable but the browser capability is, the agent can
search through the browser. Same sanitising rule.

## Fallback

Pinned `none`: skip and record `Web search: skipped by project policy`.

Say that the question could not be researched, and record it as an open
question in the document rather than answering from memory as if it were
verified.

## Project override

`CLAUDE.md`, `## Workflow`, `### Tools`: `Web search: none` for repositories where
nothing may leave the machine. Honor it absolutely.
