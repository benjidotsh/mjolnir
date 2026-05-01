---
name: using-git-worktrees
description: Use when invoked via /mjolnir:new or when the human partner explicitly asks for an isolated git worktree for feature work. Creates a worktree at .mjolnir/worktrees/<branch>/ relative to the current working tree, with no location-selection questions.
user-invocable: false
---

# Using Git Worktrees

## Overview

Git worktrees let you work on a separate branch in an isolated checkout without disturbing your current workspace. Mjölnir uses them as the default isolation primitive for non-trivial feature work.

**Core principle:** the worktree is anchored at `<current-working-tree>/.mjolnir/worktrees/<branch>/`. There is no "where should I put it?" question — the location is fixed. If you're already in a worktree, the new one nests under it; git treats them all as peers regardless of on-disk hierarchy.

**Announce at start:** "I'm using the using-git-worktrees skill to set up an isolated workspace."

## Inputs

The caller (typically `/mjolnir:new`, sometimes the human partner directly) provides a branch name. Sanitize it: lowercase, spaces → `-`, strip anything outside `[a-z0-9_/-]`.

If invoked by `/mjolnir:new`, consent has already been given — do not ask again. If invoked directly, the human partner asked for it — do not ask again. There is no y/n prompt in this skill.

## Steps

### 1. Compute the target path

```bash
WORKTREE_ROOT=$(git rev-parse --show-toplevel)
BRANCH=<sanitized-branch-name>
TARGET="$WORKTREE_ROOT/.mjolnir/worktrees/$BRANCH"
```

`WORKTREE_ROOT` is the *current* working tree — main checkout if you're in main, the worktree's root if you're in one. The new worktree nests beneath whichever you're in.

### 2. Ensure `.mjolnir/` is gitignored — and commit it before branching

The new worktree branches off the parent's current commit, so the `.mjolnir/` ignore rule must already be in that commit or the new worktree will see its own `.mjolnir/` directory as untracked.

First, probe whether the rule is already present:

```bash
cd "$WORKTREE_ROOT"
grep -qE '^/?\.mjolnir/?$' .gitignore 2>/dev/null
```

If `grep` exits 0 (rule already present), skip the rest of this step entirely — no notice, no commit, nothing to do.

If `grep` exits non-zero (rule missing), tell the human partner first, in one line, so the commit doesn't appear out of nowhere:

> "Adding `.mjolnir/` to `.gitignore` and committing it on your current branch `<current-branch>` so the new worktree inherits the rule."

Then append the rule and commit it on the parent branch:

```bash
echo '.mjolnir/' >> .gitignore
git add .gitignore
git commit -m "chore: ignore .mjolnir/ workspace directory"
```

The rule has no leading slash on purpose — it then matches `.mjolnir/` at every working-tree root, including the new worktree's own `.mjolnir/` for its specs and plans.

This is the one exception to Mjölnir's "never commit on the human partner's behalf" rule: the commit is mechanically required for the next step (creating the worktree) to produce a clean checkout, the diff is one line in `.gitignore`, and the human partner is informed before it happens.

### 3. Create the worktree

```bash
mkdir -p "$WORKTREE_ROOT/.mjolnir/worktrees"
git worktree add "$TARGET" -b "$BRANCH"
```

If the branch already exists locally, drop the `-b` flag:

```bash
git worktree add "$TARGET" "$BRANCH"
```

### 4. Run project setup (auto-detected)

```bash
cd "$TARGET"
[ -f package.json ] && npm install
[ -f Cargo.toml ] && cargo build
[ -f requirements.txt ] && pip install -r requirements.txt
[ -f pyproject.toml ] && (poetry install 2>/dev/null || pip install -e . 2>/dev/null)
[ -f go.mod ] && go mod download
```

Skip silently if nothing matches. Don't run a test baseline — that's a check the human partner can ask for explicitly if they care; running it by default adds latency for unclear value.

### 5. Report

Output the absolute path so the caller (or the human partner) knows where to operate next:

```
Worktree ready at <absolute-path>
Operating root for this flow: <absolute-path>
```

## Common Mistakes

### Anchoring at the wrong working tree

`WORKTREE_ROOT` should come from `git rev-parse --show-toplevel` (current working tree), not `git rev-parse --git-common-dir` (always main's `.git`). If you're already in a worktree and you use the wrong command, the new worktree lands in main's `.mjolnir/worktrees/` instead of nesting.

### Re-asking for consent

The skill never asks "create a worktree?" — that decision lives upstream (in `/mjolnir:new` or in the human partner's explicit request). Asking again is friction.

### Committing on behalf of the human partner

Only the `.gitignore` line for `.mjolnir/` is committed automatically by this skill, and only when it was just appended — see Step 2 for the rationale and the user-notice rule. Don't commit anything else this skill touches.

## Integration

**Called by:**
- The `/mjolnir:new` slash command — at the start of a flow when worktree isolation is requested.
- The human partner directly — when they ask for an isolated workspace.

**No cleanup skill ships with Mjölnir.** When the work is done, the human partner manages cleanup themselves (`git worktree remove <path>` or by deleting the directory and running `git worktree prune`).
