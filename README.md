# flush

End-of-session handoff skill. When the session is going down, `/flush` updates the repo with the current project state, context, and useful project memory, then commits and pushes so work can continue from another machine or agent session. AI-agnostic *output* - the repo docs work for any future agent or human reader.

## Why

The repo is the source of truth. Agent-private memory and session context are temporary: they may not survive a tool switch, machine switch, compaction, or session loss. Anything project-specific that matters later has to land in checked-in files before the session goes away.

This is a generalization of the rule "persist project-specific knowledge in project artifacts, not in any agent-private memory store."

## What `/flush` does

1. **Orient** by reading the repo state, dirty tree, and existing handoff docs.
1. **Decide what matters** from current progress, context, and project memory.
1. **Update the right files** (`AGENTS.md`, `PROGRESS.md`, `TODO.md`, `README.md`, docs, or project-specific equivalents) without creating ceremonial state.
1. **Commit and push** the files needed for handoff. Invoking `/flush` is authorization for the handoff commit/push in this solo-repo workflow.

## AI-agnostic output

The skill file itself is a Claude Code `SKILL.md` (that's the only vehicle for `user-invocable` skills today), but the body instructs the running agent in generic terms - current project state, project memory, repo handoff - so the workflow ports to any agent with a similar shape. Crucially, what the skill *writes to the repo* contains no tool-specific paths or instructions. A future contributor can read the repo and pick up.

## Install

To install this skill, ask your AI agent to symlink it into its skills directory. You can paste the following prompt:

> **Prompt:** "Please install the skill located in `./flush` by creating a symlink to it in the directory where you load global user skills."

Once installed, `/flush` or its equivalent will be available in your sessions.

## Requirements

- Git, for repo detection and commit/push (the skill asks before initializing a repo).
- `gh` (optional) for bootstrapping a fresh repo with no remote: the skill offers `gh repo create --source=. --push`, asking for visibility first.
- HTTPS pushes to github.com require `gh auth setup-git` to be run once, or another credential helper. The skill never tries to fix auth on its own - it reports and stops.

## Guarantees

- **AI decides, repo remains source of truth.** The agent chooses the smallest useful update that lets work resume.
- **No invented progress.** If there is no meaningful state to preserve, the skill says so and skips the commit.
- **No user-personal leakage.** User-scope or global memory does not go into the repo unless explicitly requested.
- **No blind staging.** The commit includes only files that should travel to the next machine.
- **No automatic cache wipe.** `/flush` preserves the handoff in git; deleting agent-private cache is a separate explicit request.
