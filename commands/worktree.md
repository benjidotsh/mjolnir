---
description: Mjölnir entry — create a worktree for this work, then route to the appropriate skill.
argument-hint: <description of the work>
---

# /mjolnir:worktree

The human partner opted into the Mjölnir workflow with explicit worktree isolation. Treat that as informed consent — don't ask whether to create a worktree.

The work they described: **$ARGUMENTS**

## Step 1 — Guards

If `$ARGUMENTS` is empty, ask the human partner what they want to work on before doing anything else.

If the current working directory is not inside a git repository, abort with: `Mjölnir's storage model assumes a git repo (`.mjolnir/` is gitignored at the repo root). Run this from inside a repo, or initialize one first.`

## Step 2 — Sanitize `$ARGUMENTS` into a branch name

Sanitize: lowercase, spaces → `-`, strip anything outside `[a-z0-9_/-]`.

## Step 3 — Create the worktree

Invoke the `mjolnir:using-git-worktrees` skill with the sanitized branch name. The skill creates the worktree at `<current-cwd>/.mjolnir/worktrees/<branch>/` and returns the absolute path. **Operating root** = that path.

## Step 4 — Anchor the operating root and route

Echo: `Operating root for this flow: <absolute-path>. All .mjolnir/specs/, .mjolnir/plans/, etc. resolve under that path.`

Then invoke the `mjolnir:routing` skill with `{ operating_root: <absolute-path>, description: $ARGUMENTS }`.

## Notes

- This command does not produce code, plans, or specs. It scaffolds isolation + operating root, then hands off. The actual work happens inside the routed skill.
