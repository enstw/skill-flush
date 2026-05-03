# skill-jz

A personal collection of AI-agent skills. Each top-level directory is one skill; the `SKILL.md` inside is the source of truth.

## Skills

- **[flush](./flush/)** — End-of-session project handoff. Updates the repo with current project state, commits, and pushes so work can resume from another machine or session.
- **[init](./init/)** — Initialize a directory with AI-agnostic agent context: `AGENTS.md` as the canonical file, plus short pointer files (`CLAUDE.md`, `GEMINI.md`, ...) that read from it. Description-triggered (no slash command), so it doesn't collide with Claude Code's built-in `/init`.

## AI-agnostic output

The `SKILL.md` format itself is Claude Code-specific — that's the only vehicle for `user-invocable` skills today — but each skill body instructs the running agent in generic terms, and what the skills *write into user repos* is agent-neutral. The workflow ports to any agent with a similar shape, and the repos these skills touch stay portable across tools and human readers.

## Install

To install a skill, ask your AI agent to symlink the skill folder into the directory where it loads global user skills. For example:

> **Prompt:** "Please install the skill located in `./flush` by creating a symlink to it in the directory where you load global user skills."

Replace `./flush` with the skill folder you want. Once installed, the skill (e.g. `/flush`) is available in your sessions.

## Requirements

Per skill — see each `SKILL.md`. In general:

- Git, for repo detection and any commit/push step a skill performs.
- `gh` (optional) for repo bootstrapping flows.
- HTTPS pushes to github.com require `gh auth setup-git` to be run once, or another credential helper. Skills never try to fix auth on their own — they report and stop.

## Adding a new skill

One skill per top-level folder. Create `<name>/SKILL.md` with frontmatter and body, link it from this README and from `AGENTS.md`, and follow the AI-agnostic-output rule for anything the skill writes into user repos.
