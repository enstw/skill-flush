# AGENTS.md

Personal collection of AI-agent skills. One folder per skill; each `SKILL.md` is the source of truth.

## Skills

- `flush/SKILL.md` — end-of-session project handoff: update repo state, commit, push.
- `init/SKILL.md` — initialize a directory with AI-agnostic agent files (`AGENTS.md` + `GEMINI.md` pointer).

## Conventions

- **AI-agnostic output.** Skill bodies and the docs they write into user repos use agent-neutral language. Don't bake tool-specific paths or instructions into a skill's body. Frontmatter (`allowed-tools`, `user-invocable`, etc.) is necessarily Claude Code-specific — that's the only Claude-specific surface.
- **One skill per top-level folder.** Add new skills as `<name>/SKILL.md`. Link them from `README.md` and from this file.
- **Solo-repo workflow.** Direct commits and pushes. No PR step.
- **Handoff-first for `/flush`.** Let the agent decide what project state matters. Don't create docs or commits just to record that nothing happened.
- **No automatic cache wipe in `/flush`.** It's a durable handoff workflow, not a cache deletion routine.

## Layout

- `flush/`, `init/`, ... — one folder per skill.
- `README.md` — outward-facing description and install instructions.
- `AGENTS.md` — this file (orientation for any agent working on the repo).
- `TODO.md` — open items.

## Install

See `README.md` for the AI-agnostic installation prompt. To install one skill, symlink its folder into the agent's global-skills directory.
