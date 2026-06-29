# tdd-executor

`tdd-executor` is a [Claude Code](https://claude.com/claude-code) **subagent** that executes an
already-approved implementation plan in strict test-driven style — writing each test first,
watching it fail, writing the minimum code to make it pass, and only then moving on.

It is an **executor, not a designer**. It does not debate architecture or invent solutions; it
turns a reviewed plan into working, tested code with mechanical discipline. It is fully
framework-agnostic — it discovers each project's test runner, file layout, fixtures, and
conventions rather than assuming a stack.

## Why a dedicated TDD executor?

LLMs left to "just implement it" tend to write production code first, skip the failing-test step,
over-build beyond the requirement, and quietly edit tests until they go green. This agent enforces
the opposite:

1. **Red before green, every cycle.** A test must be observed failing for the right reason before
   any production code is written.
2. **Minimum code to pass.** No speculative error handling, no "nice to haves," no scope creep
   beyond the current test.
3. **Tests are sacred.** It fixes the implementation to satisfy a test — never the reverse — unless
   the test itself is genuinely wrong, in which case it explains why first.
4. **One cycle at a time.** No batching three tests then three implementations; each
   red-green-refactor cycle completes and is verified before the next begins.
5. **Regression checks built in.** It runs the broader suite after each green phase, not just the
   single new test.

## How it works

The agent reads an approved plan from the conversation context (inline, or referenced from a prior
turn) and executes its steps in order. For each step it runs the **red → green → refactor** cycle,
reports progress concisely, and — when everything passes — stages only the files it changed and
commits locally (never `git add .`, never plan files, no push).

### Plan formats

It understands plain prose plans and also plans authored with the `html-planning` skill's semantic
`<plan-*>` Web Component markup (`<plan-diff>`, `<plan-file>`, `<plan-section>`, `<plan-callout>`,
etc.), which it reads as raw HTML source — those elements *are* the instructions, not decoration.
The `html-planning` integration is optional; the agent works fine with plans in any format.

## Persistent memory

The agent uses a project-scoped Persistent Agent Memory directory (`memory: project` in the
frontmatter) at `.claude/agent-memory/tdd-executor/`, committed alongside the repo so learnings are
shared with the team. It records stable, repository-specific knowledge (confirmed conventions,
recurring gotchas, test-infrastructure quirks) and deliberately avoids session-specific or
in-progress state. Switch the `memory:` flag to `local` (per-repo, gitignored) or `user`
(cross-project) if that fits your workflow better — or remove it entirely to run stateless.

## Installation

`tdd-executor` ships as a plugin in the [`hjc/agents`](../../) marketplace. From inside Claude Code:

```
/plugin marketplace add hjc/agents
/plugin install tdd-executor@agents
```

Once installed, dispatch to it after a plan is reviewed and approved — e.g. "the plan looks good,
execute it" — or invoke it explicitly.

Prefer to manage it by hand? The definition is a standard subagent file at
[`agents/tdd-executor.md`](agents/tdd-executor.md) — copy it into any `.claude/agents/` directory
Claude Code reads (project-scoped `.claude/agents/` or user-scoped `~/.claude/agents/`).

## Customization

The agent file is plain Markdown with YAML frontmatter. Common tweaks:

- **`model`** — defaults to `sonnet`. Use a more capable model for harder implementations.
- **`tools`** — ships with Playwright MCP browser tools enabled for UI verification; trim them if
  you don't need browser automation.
- **Commit conventions** — the commit step reflects a particular workflow (Co-Authored-By trailer,
  never committing `.claude/plans/`); adjust to match yours.
- **Scratch directory** — defaults to `.tmp/` for redirected test output; change if your project
  uses a different convention.

## Companion

Pairs naturally with [`git-ops`](../git-ops), a subagent dedicated to source-control
operations.

## License

Copyright (C) 2026 Hayden Chudy

Licensed under the **GNU Lesser General Public License v3.0** (LGPLv3). You are free to use this
agent in any stack, including proprietary or closed-source ones — only modifications to the files
in *this* repository must themselves be released under the LGPLv3 (or GPLv3). See
[`LICENSE`](../../LICENSE) for the LGPLv3 terms (the additional permissions) and
[`COPYING`](../../COPYING) for the GPLv3 base license they extend.
