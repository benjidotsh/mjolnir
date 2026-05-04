# Mjölnir

*Agentic development workflow based on [obra/superpowers](https://github.com/obra/superpowers)*

Three entry commands — `/mjolnir:worktree`, `/mjolnir:branch`, `/mjolnir:current` — engage a brainstorm → plan → execute → review pipeline backed by 12 skills and one code-reviewer subagent. Each command declares its isolation strategy up front (new worktree, new branch, or current state) and routes to the appropriate skill.

## Install

Inside Claude Code:

```text
/plugin marketplace add benjidotsh/mjolnir
/plugin install mjolnir@mjolnir
```

Verify with `/plugin` — `mjolnir` should appear under **Installed**. Skills are namespaced as `mjolnir:<name>`.

Update later with `/plugin marketplace update mjolnir`. Uninstall with `/plugin uninstall mjolnir@mjolnir`.

## Usage

Pick the entry command that matches your isolation preference:

```text
/mjolnir:worktree add a CLI flag to skip the cache on cold starts
/mjolnir:branch add a CLI flag to skip the cache on cold starts
/mjolnir:current add a CLI flag to skip the cache on cold starts
```

| Command | Isolation |
|---|---|
| `/mjolnir:worktree <desc>` | Creates a new worktree under `.mjolnir/worktrees/<branch>/` (sub-worktree if you're already in one) |
| `/mjolnir:branch <desc>` | `git checkout -b <branch>` in the current working tree (with structured stash/discard/abort handling for dirty trees) |
| `/mjolnir:current <desc>` | No isolation — proceeds on whatever branch and worktree are currently checked out |

After isolation, all three commands:

1. Set the operating root (the worktree's absolute path, or your current checkout).
2. Route to the right skill: brainstorming for vague intents, writing-plans for clear specs, systematic-debugging for bugs, etc.
3. From there, the engaged skill carries the flow A → Z, writing specs/plans to `<operating-root>/.mjolnir/` (gitignored, never committed).

## Design choices vs. superpowers

| Choice | Mjölnir | Superpowers |
|---|---|---|
| Activation | Opt-in via `/mjolnir:worktree`, `/mjolnir:branch`, or `/mjolnir:current` | Auto-injected on every SessionStart |
| Slash commands | Three explicit entries (`/mjolnir:worktree`, `/mjolnir:branch`, `/mjolnir:current`) | Three (deprecated, redirect to skills) |
| Specs/plans location | `<operating-root>/.mjolnir/` (gitignored) | `docs/superpowers/` (committed) |
| Worktrees | Hierarchical (sub-worktrees allowed); fixed at `.mjolnir/worktrees/<branch>/`; opt in by invoking `/mjolnir:worktree` directly | Asks where each invocation |
| Execution | Subagent-driven only | Subagent-driven + inline-execution alternatives |
| Branch finishing | Hand back to human partner | `finishing-a-development-branch` skill drives merge/PR |
| `settings.json` | Not shipped (yours stays authoritative) | Not shipped |

## Skills

| Skill | Phase |
|---|---|
| `mjolnir:brainstorming` | design |
| `mjolnir:writing-plans` | plan |
| `mjolnir:subagent-driven-development` | execute |
| `mjolnir:dispatching-parallel-agents` | execute (parallel) |
| `mjolnir:test-driven-development` | execute |
| `mjolnir:systematic-debugging` | quality |
| `mjolnir:verification-before-completion` | quality |
| `mjolnir:requesting-code-review` | review |
| `mjolnir:receiving-code-review` | review |
| `mjolnir:using-git-worktrees` | workflow |
| `mjolnir:routing` | workflow |
| `mjolnir:writing-skills` | meta |

## License

MIT. Inherits attribution from superpowers — see `LICENSE`.
