---
description: Mjölnir entry — proceed on the current branch and worktree, then route to the appropriate skill.
argument-hint: <description of the work>
---

# /mjolnir:current

The human partner opted into the Mjölnir workflow without isolation — work proceeds on whatever branch and worktree are currently checked out.

The work they described: **$ARGUMENTS**

## Step 1 — Guards

If `$ARGUMENTS` is empty, ask the human partner what they want to work on before doing anything else.

If the current working directory is not inside a git repository, abort with: `Mjölnir's storage model assumes a git repo (`.mjolnir/` is gitignored at the repo root). Run this from inside a repo, or initialize one first.`

## Step 2 — Anchor the operating root and route

Operating root = `git rev-parse --show-toplevel`.

Echo: `Operating root for this flow: <absolute-path>. All .mjolnir/specs/, .mjolnir/plans/, etc. resolve under that path.`

Invoke the `mjolnir:routing` skill with `{ operating_root: <absolute-path>, description: $ARGUMENTS }`.

## Notes

- No isolation work happens here. Use this when you've already set up the branch/worktree the way you want and just need Mjölnir's routing.
- This command does not produce code, plans, or specs. The actual work happens inside the routed skill.
