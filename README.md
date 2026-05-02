# shutdown

End-of-session skill that migrates the running agent's project-specific memory into the repo's living state docs (`AGENTS.md`, `PROGRESS.md`, `TODO.md`, …), commits and pushes the changes, then wipes the agent's memory for this project. AI-agnostic *output* — the docs work for any future agent or human reader.

## Why

The repo is the source of truth. Agent-private memory is a cache: it won't survive a tool switch, a wipe, or a coworker. Anything project-specific has to land in checked-in files before the cache goes away.

This is a generalization of the rule "persist project-specific knowledge in project artifacts, not in any agent-private memory store."

## What `/shutdown` does

1. **Migrate** project-scope entries from per-project agent memory into `AGENTS.md` / `PROGRESS.md` / `TODO.md`. User-scope memories (preferences, role, who-the-user-is) stay in agent memory.
1. **Add session state** — anything new from the current conversation that isn't in memory or the docs yet.
1. **Commit and push** the doc changes. Invoking `/shutdown` is authorization. Solo-repo workflow — no PR step.
1. **Wipe** the agent's per-project memory for this project, after explicit confirmation. Mandatory — keeping project memory around defeats the purpose.

## AI-agnostic output

The skill file itself is a Claude Code `SKILL.md` (that's the only vehicle for `user-invocable` skills today), but the body instructs the running agent in generic terms — "your own per-project memory", "your agent memory" — so the workflow ports to any agent with a similar shape. Crucially, what the skill *writes to the repo* contains no tool-specific paths or instructions. A future contributor (agent or human, Claude or otherwise) can read the docs and pick up.

## Install

Symlink or copy `SKILL.md` into a directory your harness loads skills from. For Claude Code:

```sh
mkdir -p ~/.claude/skills/shutdown
ln -s "$(pwd)/SKILL.md" ~/.claude/skills/shutdown/SKILL.md
```

Then `/shutdown` is available.

## Requirements

- Git, for repo detection and commit/push (skill skips git steps gracefully if the cwd isn't a repo).
- `gh` (optional) for bootstrapping a fresh repo with no remote: the skill offers `gh repo create --source=. --push`, asking for visibility first.
- HTTPS pushes to github.com require `gh auth setup-git` to be run once, or another credential helper. The skill never tries to fix auth on its own — it reports and stops.

## Guarantees

- **Wipe is this project only.** Other projects' memory and any user-scope/global memory are untouched.
- **Confirmation gate.** The wipe shows the exact command and waits for explicit approval.
- **Push failures don't block the wipe.** The skill reports and continues.
- **No `git add -A`.** The commit only includes the docs the skill updated, in case you have unrelated working-tree changes.
