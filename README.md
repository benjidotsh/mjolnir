# Mjölnir

*Agentic development workflow based on [obra/superpowers](https://github.com/obra/superpowers)*

One command — `/mjolnir:new <description>` — engages a brainstorm → plan → execute → review pipeline backed by 11 skills and one code-reviewer subagent.

## Install

Inside Claude Code:

```text
/plugin marketplace add benjidotsh/mjolnir
/plugin install mjolnir@mjolnir
```

Verify with `/plugin` — `mjolnir` should appear under **Installed**. Skills are namespaced as `mjolnir:<name>`.

Update later with `/plugin marketplace update mjolnir`. Uninstall with `/plugin uninstall mjolnir@mjolnir`.

## Usage

```text
/mjolnir:new add a CLI flag to skip the cache on cold starts
```

Mjölnir then:

1. Detects whether you're in main or already in a worktree.
2. Asks whether to create a (sub-)worktree for this work.
3. Sets the operating root (the worktree's absolute path, or your current checkout).
4. Routes to the right skill: brainstorming for vague intents, writing-plans for clear specs, systematic-debugging for bugs, etc.
5. From there, the engaged skill carries the flow A → Z, writing specs/plans to `<operating-root>/.mjolnir/` (gitignored, never committed).

## Design choices vs. superpowers

| Choice | Mjölnir | Superpowers |
|---|---|---|
| Activation | Opt-in via `/mjolnir:new` | Auto-injected on every SessionStart |
| Slash commands | One (`/mjolnir:new`) | Three (deprecated, redirect to skills) |
| Specs/plans location | `<operating-root>/.mjolnir/` (gitignored) | `docs/superpowers/` (committed) |
| Worktrees | Hierarchical (sub-worktrees allowed); fixed at `.mjolnir/worktrees/<branch>/`; one y/n question owned by `/mjolnir:new` | Asks where each invocation |
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
| `mjolnir:writing-skills` | meta |

## License

MIT. Inherits attribution from superpowers — see `LICENSE`.
