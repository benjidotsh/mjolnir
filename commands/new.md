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

Always ask, regardless of where you are. The phrasing depends on context:

- In main: **"Create a worktree for this work? (y/n)"**
- In a worktree: **"Create a sub-worktree for this work? (y/n — 'n' = continue in the current worktree)"**

Wait for the answer. Don't infer.

- On `y`: invoke the `mjolnir:using-git-worktrees` skill. Pass it a branch name derived from `$ARGUMENTS` (sanitize: lowercase, spaces → `-`, strip non-`[a-z0-9_/-]`). The skill creates the worktree at `<current-cwd>/.mjolnir/worktrees/<branch>/` and returns the absolute path. **Operating root** = that path.
- On `n`: **operating root** = the current `--show-toplevel`.

## Step 3 — Anchor the operating root, then route

Before invoking any other skill, state the operating root explicitly so the rest of the flow can resolve paths:

> "Operating root for this flow: `<absolute-path>`. All `.mjolnir/specs/`, `.mjolnir/plans/`, etc. resolve under that path."

Then route based on what `$ARGUMENTS` describes. Pick one — don't fan out:

| If the human partner is describing... | Engage |
|---|---|
| a vague intent, a goal, no spec yet | `mjolnir:brainstorming` |
| a clear spec or written requirements, ready to plan | `mjolnir:writing-plans` |
| an existing plan they want executed | `mjolnir:subagent-driven-development` |
| a bug, a test failure, an unexpected behavior | `mjolnir:systematic-debugging` |
| a feature they want built test-first | `mjolnir:test-driven-development` |
| code they want reviewed | `mjolnir:requesting-code-review` |
| review feedback they're responding to | `mjolnir:receiving-code-review` |
| 2+ independent things to investigate in parallel | `mjolnir:dispatching-parallel-agents` |

If `$ARGUMENTS` is ambiguous between options, ask the human partner one short clarifying question before routing — don't guess.

## Notes on this command

- `/mjolnir:new` does not itself produce code, plans, or specs. It scaffolds the operating root and hands off. The actual work happens inside the routed skill.
- If the human partner invokes `/mjolnir:new` from outside a git repo, abort with a clear message: Mjölnir's storage model assumes a repo (`.mjolnir/` is gitignored at the repo root).
- If `$ARGUMENTS` is empty, ask the human partner what they want to work on before doing any of the above.
