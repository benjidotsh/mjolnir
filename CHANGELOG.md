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

## [0.1.0] - 2026-05-01

Initial release. Mjölnir is an opt-in agentic development workflow derived from [obra/superpowers](https://github.com/obra/superpowers), reshaped for explicit invocation, gitignored work artifacts, and subagent-only execution.

### Added

- `/mjolnir:new <description>` — single opt-in entry command. Detects worktree state, prompts via `AskUserQuestion` whether to create a (sub-)worktree, sets the operating root, and routes to the appropriate skill.
- 11 skills covering the full design → plan → execute → review pipeline:
  - `mjolnir:brainstorming` — turn ideas into specs at `<operating-root>/.mjolnir/specs/`.
  - `mjolnir:writing-plans` — turn specs into bite-sized implementation plans at `<operating-root>/.mjolnir/plans/`.
  - `mjolnir:subagent-driven-development` — execute plans task-by-task with a fresh subagent per task and two-stage review (spec then quality) between tasks.
  - `mjolnir:dispatching-parallel-agents` — fan out independent investigations to parallel subagents.
  - `mjolnir:test-driven-development` — RED → GREEN → REFACTOR → commit cycle.
  - `mjolnir:systematic-debugging` — 4-phase root-cause discovery before any fix.
  - `mjolnir:verification-before-completion` — evidence-before-assertions gate on completion claims.
  - `mjolnir:requesting-code-review` — dispatch the `mjolnir:code-reviewer` subagent with crafted context.
  - `mjolnir:receiving-code-review` — restate, verify, evaluate, implement — one item at a time.
  - `mjolnir:using-git-worktrees` — fixed-location worktrees at `<current-working-tree>/.mjolnir/worktrees/<branch>/`.
  - `mjolnir:writing-skills` — TDD applied to skill authoring.
- `mjolnir:code-reviewer` subagent — invoked by `requesting-code-review` and by `subagent-driven-development`'s per-task code-quality gate.
- Marketplace metadata under `.claude-plugin/` for installation via `/plugin marketplace add benjidotsh/mjolnir`.

### Design choices vs. superpowers

- Opt-in activation (single slash command) rather than auto-injection on `SessionStart`.
- Specs and plans saved under `<operating-root>/.mjolnir/` and gitignored — meta-files never enter the project repo.
- Hierarchical worktrees fixed at `.mjolnir/worktrees/<branch>/`; sub-worktrees allowed when invoking from inside an existing worktree.
- Subagent-driven execution only — no inline-execution alternatives ship.
- Branch finishing left to the human partner — Mjölnir does not drive merge/PR/cleanup flows.
- No `settings.json` shipped — user settings remain authoritative.
