---
name: init-agnostic
description: Initialize a directory with AI-agnostic agent context. Canonical project context lives in AGENTS.md; agent-specific files (CLAUDE.md, GEMINI.md, ...) become short pointer files that read from AGENTS.md. Use when the user asks to set up AI-agnostic agent files for a project, or to migrate single-agent context into the agnostic pattern. Distinct from Claude Code's built-in /init, which writes CLAUDE.md directly.
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash(pwd)
  - Bash(ls *)
  - Bash(git rev-parse --show-toplevel)
  - Bash(git status *)
  - Bash(git diff *)
  - Bash(git add *)
  - Bash(git commit *)
---

# /init-agnostic - AI-agnostic agent context

Canonical project context for AI agents lives in `AGENTS.md`. Agent-specific files (`CLAUDE.md`, `GEMINI.md`, and friends) become short pointer files that send the agent to `AGENTS.md`. The repo stays portable across tools and human readers, and there is one source of truth to maintain.

This skill does not clash with Claude Code's built-in `/init` (which writes `CLAUDE.md` directly). Use this skill when you want the agnostic pattern instead.

## Flow

1. **Locate root.** Run `git rev-parse --show-toplevel` if available, otherwise use the current working directory and confirm with the user.
1. **Survey existing context files** in the root: `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `.cursorrules`, `.cursor/rules/`, `.github/copilot-instructions.md`, and any project-specific equivalents. Read enough to know which have substantive content.
1. **Confirm the agent set with the user.** Default pointer set: `CLAUDE.md` and `GEMINI.md`. Ask whether to add or remove pointers (`.cursorrules`, `.github/copilot-instructions.md`, etc.) before writing anything. Do not assume.
1. **Decide canonical content for `AGENTS.md`.**
   - If `AGENTS.md` already exists with content, treat it as canonical and leave its body alone.
   - Otherwise, if exactly one of the per-agent files (`CLAUDE.md`, `GEMINI.md`, ...) has substantive project content, move that file's content into a new `AGENTS.md` (the per-agent file then becomes a pointer in the next step).
   - Otherwise, if multiple per-agent files have substantive content, stop and ask the user how to merge - do not silently pick one.
   - Otherwise (nothing exists), create a fresh `AGENTS.md` with section stubs: Project Overview, Setup, Architecture, Conventions, Commands.
1. **Create pointer files** for each agent in the confirmed set. Each pointer is short and looks like:

   ```markdown
   # AI Agent Context

   The contextual instructions for AI agents working in this repository are in [AGENTS.md](./AGENTS.md).

   Read `AGENTS.md` for project overview, architecture, conventions, and commands.
   ```

   If a pointer file already exists with substantive (non-pointer) content, do not overwrite it. Stop and ask the user how to merge into `AGENTS.md` first.
1. **Report and offer to commit.** Run `git status` to show what changed. Suggest a commit message like `chore: switch to AI-agnostic agent context (AGENTS.md + pointers)` and ask before staging/committing. Do not auto-commit.

## Guarantees

- **No silent overwrites.** If a per-agent file already has real content, the skill stops and asks before merging.
- **Agent set is user-confirmed.** The default is `CLAUDE.md` + `GEMINI.md`, but the user can add or remove pointers before any file is written.
- **No commits without permission.** The skill stages and commits only after the user confirms.
- **Agent-neutral output.** What the skill writes into the user's repo - `AGENTS.md` body and the pointer files - contains no skill-internal or tool-specific instructions beyond naming the file each agent reads.
