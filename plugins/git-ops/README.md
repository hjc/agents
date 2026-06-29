# git-ops

`git-ops` is a [Claude Code](https://claude.com/claude-code) **subagent** that handles all
source-control operations — committing, branching, pushing/pulling, stashing, merging, rebasing,
cherry-picking, tagging, and history inspection — as a dedicated, disciplined specialist.

It is framework- and project-agnostic: it carries no knowledge of any particular stack and
discovers each repository's conventions (default branch, commit-message style, branch naming) as
it works.

## Why a dedicated git agent?

Git is high-stakes and easy to get subtly wrong: a stray `git add .` sweeps in unrelated work, a
careless `--force` rewrites published history, a missed generated file breaks a build. Delegating
source control to a single specialized agent gives you:

1. **Surgical staging.** It never blindly `git add .` — it stages only the files relevant to the
   described change and reports anything left behind.
2. **Verification after every step.** It confirms the result of each operation (line counts after
   copies, `git show --stat` after commits, `git diff --cached --name-only` after staging) instead
   of assuming success.
3. **Concise, evidence-based status reports.** Every operation comes back as a 1–3 line summary
   with the real commit hash, branch, and any warnings — observed facts, not intentions.
4. **Guardrails by default.** It refuses to force-push or rewrite published history without
   explicit approval, never runs `git clean`, never deletes files, and never commits planning
   artifacts under `.claude/plans/`.
5. **A smaller blast radius on your main context.** Noisy git output stays in the subagent.

## Behavior highlights

- **Conventional commits** (`feat:`, `fix:`, `chore:`, `docs:`, `ai:`) with messages that describe
  code changes only — never plan names, phase numbers, or ticket IDs.
- **Atomic commit sequencing** — large changes are split into discrete, purpose-scoped commits.
- **Generated companion files** (migrations, lockfiles, generated clients) are detected and
  committed alongside the source change that produced them.
- **Merge conflicts are delegated**, not guessed at — it hands conflicting files off to a
  code-implementation agent (such as the companion [`tdd-executor`](../tdd-executor) agent)
  rather than resolving them blindly.
- **Repository hygiene** — prunes stale remote-tracking refs, sets up tracking on new branches,
  and refuses to delete protected branches (`main`/`master`/`develop`).

## Persistent memory

The agent uses a project-scoped memory directory (`memory: project` in the frontmatter) at
`.claude/agent-memory/git-ops/`, committed alongside the repo so its observations are shared with
the team. It records repository-specific git conventions it discovers — branch naming, default
branch, merge-vs-rebase preference, protected branches, CI triggers — so later sessions don't
re-learn them. Switch the `memory:` flag to `local` (per-repo, gitignored) or `user`
(cross-project) if that fits your workflow better — or remove it entirely to run stateless.

## Installation

`git-ops` ships as a plugin in the [`hjc/agents`](../../) marketplace. From inside Claude Code:

```
/plugin marketplace add hjc/agents
/plugin install git-ops@hjc-agents
```

Once installed, Claude Code dispatches to it automatically when you ask to commit, branch, push,
merge, or otherwise work with source control — or you can invoke it explicitly.

Prefer to manage it by hand? The definition is a standard subagent file at
[`agents/git-ops.md`](agents/git-ops.md) — copy it into any `.claude/agents/` directory Claude Code
reads (project-scoped `.claude/agents/` or user-scoped `~/.claude/agents/`).

## Customization

The agent file is plain Markdown with YAML frontmatter. Common tweaks:

- **`model`** — defaults to `haiku` (fast and cheap for mechanical git work). Bump to `sonnet` if
  you want more careful reasoning on complex merges.
- **Commit tags** — edit the tags table to match your team's convention.
- **`.tmp/` and `.claude/plans/` conventions** — these reflect a particular planning workflow;
  adjust or remove the references if they don't match yours.

## License

Copyright (C) 2026 Hayden Chudy

Licensed under the **GNU Lesser General Public License v3.0** (LGPLv3). You are free to use this
agent in any stack, including proprietary or closed-source ones — only modifications to the files
in *this* repository must themselves be released under the LGPLv3 (or GPLv3). See
[`LICENSE`](../../LICENSE) for the LGPLv3 terms (the additional permissions) and
[`COPYING`](../../COPYING) for the GPLv3 base license they extend.
