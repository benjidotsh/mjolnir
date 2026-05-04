---
description: Mjölnir entry — create a new branch in the current working tree, then route to the appropriate skill.
argument-hint: <description of the work>
---

# /mjolnir:branch

The human partner opted into the Mjölnir workflow with explicit new-branch isolation in the current working tree. Treat that as informed consent — don't ask whether to switch branches.

The work they described: **$ARGUMENTS**

## Step 1 — Guards

If `$ARGUMENTS` is empty, ask the human partner what they want to work on before doing anything else.

If the current working directory is not inside a git repository, abort with: `Mjölnir's storage model assumes a git repo (`.mjolnir/` is gitignored at the repo root). Run this from inside a repo, or initialize one first.`

## Step 2 — Sanitize `$ARGUMENTS` into a branch name

Sanitize: lowercase, spaces → `-`, strip anything outside `[a-z0-9_/-]`.

## Step 3 — Handle uncommitted changes

Run `git status --porcelain`. If the output is empty, the working tree is clean — skip the rest of this step.

If the output is non-empty, the working tree is dirty. Ask the human partner via the `AskUserQuestion` tool:

- `question`: `"Working tree has uncommitted changes. How should we handle them before creating the branch?"`
- `header`: `"Dirty tree"`
- `multiSelect`: `false`
- `options`:
  - `{"label": "Stash", "description": "Run git stash push -m 'mjolnir: pre-branch stash for <branch>' then proceed"}`
  - `{"label": "Discard (destructive)", "description": "Run git reset --hard HEAD && git clean -fd then proceed. This permanently deletes uncommitted changes."}`
  - `{"label": "Abort", "description": "Exit cleanly without creating the branch"}`

Wait for the answer. Don't infer.

- **Stash** → run `git stash push -m "mjolnir: pre-branch stash for <branch>"`. Continue.
- **Discard (destructive)** → run `git reset --hard HEAD && git clean -fd`. Continue.
- **Abort** → stop. Do not create the branch, do not anchor, do not route. Tell the human partner: `Aborted — no changes made.`

## Step 4 — Create the branch

Run `git checkout -b <branch>`.

## Step 5 — Anchor the operating root and route

Operating root = `git rev-parse --show-toplevel`.

Echo: `Operating root for this flow: <absolute-path>. All .mjolnir/specs/, .mjolnir/plans/, etc. resolve under that path.`

Invoke the `mjolnir:routing` skill with `{ operating_root: <absolute-path>, description: $ARGUMENTS }`.

## Notes

- This command does not produce code, plans, or specs. It scaffolds the branch + operating root, then hands off. The actual work happens inside the routed skill.
- Inside an existing worktree, this command switches *that worktree's* HEAD to the new branch. If you want a separate isolated workspace, use `/mjolnir:worktree` instead.
