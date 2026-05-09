# skill-jz

A personal collection of AI-agent skills. Each top-level directory is one skill; the `SKILL.md` inside is the source of truth.

## Skills

- **[flush](./flush/)** — End-of-session project handoff. Updates the repo with current project state, commits, and pushes so work can resume from another machine or session.
- **[init](./init/)** — Initialize a directory with AI-agnostic agent context: `AGENTS.md` as the canonical file, plus short pointers from agent-specific instruction files such as `CLAUDE.md` and `GEMINI.md`. Description-triggered (no slash command), so it doesn't collide with Claude Code's built-in `/init`.
- **[recommend](./recommend/)** — Pause the current trajectory, surface direction-level recommendations, and offer to refactor. Slash-invocable as `/recommend`; also self-triggers when the agent senses drift (scope creep, naming churn, half-finished implementations).
- **[self-evaluate](./self-evaluate/)** — Estimate how many PDCA (plan-do-check-act) loops remain before the work is finished. Cost-driven and phase-agnostic: invokable pre-implementation, mid-implementation, or post-test-failure. The agent investigates (reads code, smoke-tests, checks env, web-searches) before estimating so the number is grounded in evidence. Slash-invocable as `/self-evaluate`.

## AI-agnostic output

Some `SKILL.md` metadata is runner-specific, such as `allowed-tools` and `user-invocable`; that metadata is the only agent-specific surface. Each skill body instructs the running agent in generic terms, and what the skills *write into user repos* is agent-neutral. The workflow ports to any agent with a similar shape, and the repos these skills touch stay portable across tools and human readers.

## Install

To install one skill, ask your AI agent to symlink the skill folder into the directory where it loads global user skills. For example:

> **Prompt:** "Please install the skill located in `./flush` by creating a symlink to it in the directory where you load global user skills."

Replace `./flush` with the skill folder you want. Once installed, the skill (e.g. `/flush`) is available in your sessions.

To install all skills in this repo, ask:

> **Prompt:** "Please install all skills in this repo by symlinking each top-level folder that contains a `SKILL.md` into the directory where you load global user skills. Replace stale symlinks with the same names, but ask before overwriting any real directory or non-symlink file."

Restart or reload the agent session if the skill list is cached.

## Requirements

Per skill — see each `SKILL.md`. In general:

- Git, for repo detection and any commit/push step a skill performs.
- `gh` (optional) for repo bootstrapping flows.
- HTTPS pushes to github.com require `gh auth setup-git` to be run once, or another credential helper. Skills never try to fix auth on their own — they report and stop.

## Adding a new skill

One skill per top-level folder. Create `<name>/SKILL.md` with frontmatter and body, link it from this README and from `AGENTS.md`, and follow the AI-agnostic-output rule for anything the skill writes into user repos.
