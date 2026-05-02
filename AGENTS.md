# AGENTS.md

Claude Code skill repository. The skill is **flush** — `flush/SKILL.md` is the source of truth.

## What `/flush` does

Records an end-of-session project handoff in the repo, commits, and pushes so work can continue from another machine or agent session. Repo is durable; agent memory and session context are temporary.

## Conventions

- **AI-agnostic output.** The skill body and the docs it writes use agent-neutral language. Don't bake tool-specific paths or instructions into the skill text. The frontmatter `allowed-tools` is necessarily Claude Code-specific — that's the only Claude-specific surface.
- **Solo-repo workflow.** `/flush` commits and pushes directly. No PR step.
- **Handoff-first.** Let the agent decide what project state matters. Do not create docs or commits just to record that nothing happened.
- **No automatic cache wipe.** `/flush` is now a durable handoff workflow, not a cache deletion routine.

## Layout

- `flush/SKILL.md` — the skill.
- `README.md` — outward-facing description and install instructions.
- `AGENTS.md` — this file (orientation for any agent working on the repo).
- `TODO.md` — open items.

## Install

See `README.md` for context, requirements, and the AI-agnostic installation prompt.
