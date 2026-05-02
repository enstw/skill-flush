# AGENTS.md

Claude Code skill repository. The skill is **flush** — `flush/SKILL.md` is the source of truth.

## What `/flush` does

Migrates per-session agent project memory into the repo's state docs, commits and pushes, then wipes the agent's project memory. Repo is durable; agent memory is just cache.

## Conventions

- **AI-agnostic output.** The skill body and the docs it writes use agent-neutral language. Don't bake tool-specific paths or instructions into the skill text. The frontmatter `allowed-tools` is necessarily Claude Code-specific — that's the only Claude-specific surface.
- **Solo-repo workflow.** `/flush` commits and pushes directly. No PR step.
- **Wipe is mandatory.** There is no `--no-clear` flag — project memory must not survive past a session.

## Layout

- `flush/SKILL.md` — the skill.
- `README.md` — outward-facing description and install instructions.
- `AGENTS.md` — this file (orientation for any agent working on the repo).
- `TODO.md` — open items.

## Install

See `README.md` for context, requirements, and the AI-agnostic installation prompt.
