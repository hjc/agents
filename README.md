# agents

A [Claude Code](https://claude.com/claude-code) **plugin marketplace** distributing dedicated,
framework-agnostic software-development subagents. Each agent runs in its own context window and is
dispatched via the Task tool, keeping specialized work off your main thread.

## Plugins

| Plugin | What it does |
|--------|--------------|
| **[git-ops](plugins/git-ops)** | A disciplined source-control specialist — surgical staging, conventional commits, verified status reports, and guardrails against destructive operations. |
| **[tdd-executor](plugins/tdd-executor)** | Executes an already-approved implementation plan in strict red-green-refactor TDD style — test first, minimum code to pass, one cycle at a time. |

## Installation

From inside Claude Code, add this marketplace once, then install whichever plugins you want:

```
/plugin marketplace add hjc/agents
/plugin install git-ops@agents
/plugin install tdd-executor@agents
```

Run `/plugin marketplace update` to pull new versions when they ship, and manage installed plugins
with `/plugin`. Each plugin's own README covers its behavior, configuration, and persistent-memory
setup.

## License

Copyright (C) 2026 Hayden Chudy

Licensed under the **GNU Lesser General Public License v3.0** (LGPLv3). You are free to use these
agents in any stack, including proprietary or closed-source ones — only modifications to the files
in *this* repository must themselves be released under the LGPLv3 (or GPLv3). See
[`LICENSE`](LICENSE) for the LGPLv3 terms and [`COPYING`](COPYING) for the GPLv3 base license they
extend.
