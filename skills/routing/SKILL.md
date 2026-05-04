---
name: routing
description: Use when invoked by a Mjölnir entry command (/mjolnir:worktree, /mjolnir:branch, or /mjolnir:current) to dispatch the work to the appropriate downstream skill based on what the human partner described.
user-invocable: false
---

# Routing

## Overview

Mjölnir entry commands (`/mjolnir:worktree`, `/mjolnir:branch`, `/mjolnir:current`) call this skill once they've established the operating root. This skill owns the dispatch table — it picks one downstream skill based on what the human partner described and hands off.

**Announce at start:** "I'm using the routing skill to dispatch this work."

## Inputs

The caller passes:
- **operating_root** — absolute path. Where `.mjolnir/specs/`, `.mjolnir/plans/`, etc. resolve.
- **description** — the original work description (the entry command's `$ARGUMENTS`).

## Steps

### 1. Confirm the operating-root anchor was echoed

The entry command should already have echoed:

> "Operating root for this flow: `<absolute-path>`. All `.mjolnir/specs/`, `.mjolnir/plans/`, etc. resolve under that path."

If for any reason it wasn't echoed (e.g., this skill was invoked directly), echo it now before continuing.

### 2. Route based on the description

| If the human partner is describing... | Engage |
|---|---|
| a feature, change, or goal — and no written spec artifact exists yet | `mjolnir:brainstorming` |
| a written spec artifact already exists (a `.mjolnir/specs/` file, a linked design doc, or pasted requirements that already cover scope, constraints, and success criteria) | `mjolnir:writing-plans` |
| an existing plan they want executed | `mjolnir:subagent-driven-development` |
| a bug, a test failure, an unexpected behavior | `mjolnir:systematic-debugging` |
| a feature they want built test-first | `mjolnir:test-driven-development` |
| code they want reviewed | `mjolnir:requesting-code-review` |
| review feedback they're responding to | `mjolnir:receiving-code-review` |
| 2+ independent things to investigate in parallel | `mjolnir:dispatching-parallel-agents` |

**Default to `brainstorming` for anything design-shaped.** A confidently-phrased one-liner ("add X to Y", "refactor Z to do W", "build a thing that does Q") is *intent*, not a spec. The bar for `writing-plans` is an artifact you can point at — not a feeling that the request sounded clear. If no such artifact exists, route to `brainstorming` even when the request seems obvious. A short design pass surfaces hidden assumptions cheaply; skipping it and discovering them mid-implementation is expensive. Bug reports, code review, and executing an existing plan are not design-shaped — route those by their own rows.

If the description is genuinely ambiguous between two non-brainstorming routes (e.g., bug vs. review feedback), ask the human partner one short clarifying question before routing — don't guess.

## Notes

- The repo guard and empty-arguments guard live in the entry commands, not here. By the time this skill runs, both have already passed.
- Pick exactly one downstream skill — don't fan out.
