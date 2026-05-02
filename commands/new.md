---
description: Opt-in entry point for the Mjölnir workflow. Sets up an operating root (worktree or current checkout), then routes to the appropriate skill based on what the human partner described.
argument-hint: <description of the work>
---

# /mjolnir:new

The human partner has explicitly opted into the Mjölnir workflow for this task. Treat that as informed consent: this is heavier machinery than a one-shot answer, and they want it engaged.

The work they described: **$ARGUMENTS**

Run this orchestration in order. Don't skip steps and don't compress them — each step exists to set up the next one cleanly.

## Step 1 — Detect the current working tree

```bash
git rev-parse --show-toplevel    # current working tree root (main or worktree)
git rev-parse --git-common-dir   # main repo's .git
```

If `--show-toplevel` and `dirname --git-common-dir` differ, you're already inside a worktree. Use that fact to phrase the next question accurately ("worktree" if in main, "sub-worktree" if already in a worktree). The current working tree is the *parent* of any new worktree we create.

## Step 2 — Ask whether to create a (sub-)worktree

Always ask, regardless of where you are. Use the `AskUserQuestion` tool — never plain text — so the human partner gets a structured prompt with labeled options.

Tool call shape (phrasing varies by context):

- In main checkout — `question`: `"Create a worktree for this work?"`, `header`: `"Worktree"`, `multiSelect`: `false`, `options`: `[{"label": "Yes, create worktree", "description": "Create an isolated worktree under .mjolnir/worktrees/ for this task"}, {"label": "No, work in main", "description": "Continue working in the current main checkout"}]`
- Already in a worktree — `question`: `"Create a sub-worktree for this work?"`, `header`: `"Sub-worktree"`, `multiSelect`: `false`, `options`: `[{"label": "Yes, create sub-worktree", "description": "Nest a new worktree under the current one for this task"}, {"label": "No, continue here", "description": "Stay in the current worktree"}]`

Wait for the answer. Don't infer.

- On a "Yes" answer: invoke the `mjolnir:using-git-worktrees` skill. Pass it a branch name derived from `$ARGUMENTS` (sanitize: lowercase, spaces → `-`, strip non-`[a-z0-9_/-]`). The skill creates the worktree at `<current-cwd>/.mjolnir/worktrees/<branch>/` and returns the absolute path. **Operating root** = that path.
- On a "No" answer: **operating root** = the current `--show-toplevel`.

## Step 3 — Anchor the operating root, then route

Before invoking any other skill, state the operating root explicitly so the rest of the flow can resolve paths:

> "Operating root for this flow: `<absolute-path>`. All `.mjolnir/specs/`, `.mjolnir/plans/`, etc. resolve under that path."

Then route based on what `$ARGUMENTS` describes. Pick one — don't fan out:

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

If `$ARGUMENTS` is genuinely ambiguous between two non-brainstorming routes (e.g., bug vs. review feedback), ask the human partner one short clarifying question before routing — don't guess.

## Notes on this command

- `/mjolnir:new` does not itself produce code, plans, or specs. It scaffolds the operating root and hands off. The actual work happens inside the routed skill.
- If the human partner invokes `/mjolnir:new` from outside a git repo, abort with a clear message: Mjölnir's storage model assumes a repo (`.mjolnir/` is gitignored at the repo root).
- If `$ARGUMENTS` is empty, ask the human partner what they want to work on before doing any of the above.
