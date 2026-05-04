# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.2.0] - 2026-05-04

### Added

- `/mjolnir:worktree <description>` — entry command that creates a new worktree under `.mjolnir/worktrees/<branch>/` before routing.
- `/mjolnir:branch <description>` — entry command that creates a new branch in the current working tree before routing. Prompts via `AskUserQuestion` (Stash / Discard / Abort) when the working tree is dirty.
- `/mjolnir:current <description>` — entry command that proceeds on the current branch and worktree before routing.
- `mjolnir:routing` skill — dispatches work to the appropriate downstream skill. Owned by the three entry commands; not user-invocable.

### Removed

- `/mjolnir:new` — replaced by the three entry commands above.

### Changed

- All skill descriptions previously prefixed with `Use when invoked via /mjolnir:new` now use the generic `Use when invoked by a Mjölnir entry command` prefix. Skill bodies that referenced `/mjolnir:new` as the operating-root setter now name the three entry commands explicitly.
- `mjolnir:using-git-worktrees` skill description and integration text now reference `/mjolnir:worktree` (its specific upstream caller).

### Migration

Replace `/mjolnir:new <desc>` invocations with one of:

- `/mjolnir:worktree <desc>` — equivalent to selecting "Yes, create worktree" in the previous flow.
- `/mjolnir:branch <desc>` — new option not previously available: fresh branch in the current tree, no worktree.
- `/mjolnir:current <desc>` — equivalent to selecting "No, work in main / continue here" in the previous flow.
