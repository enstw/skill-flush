---
name: flush
description: End-of-session project handoff. Use when the session is ending, context may be lost, work needs to continue on another machine, or the user asks to save current project state. Decide what current progress, context, and project memory must be written into the repo; update the right project files; commit and push so the repo is enough to resume from.
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash(pwd)
  - Bash(date)
  - Bash(git rev-parse --show-toplevel)
  - Bash(git status *)
  - Bash(git diff *)
  - Bash(git ls-files *)
  - Bash(git branch *)
  - Bash(git remote *)
  - Bash(git add *)
  - Bash(git commit *)
  - Bash(git push)
  - Bash(git push *)
  - Bash(git init)
  - Bash(gh repo create *)
  - Bash(ls *)
  - Bash(find *)
---

# /flush - End-of-session handoff

The user is saying: this session is going down; preserve the useful project state, commit it, and push it so the next session or another machine can resume from the repo alone.

Principle: the repo is durable. Agent-private memory, transcripts, and scratch context are temporary. Do not dump the transcript; extract only the state a future contributor needs.

This skill is a handoff, not a cache deletion routine. Do not wipe agent memory, transcripts, or local caches unless the user explicitly asks for that separate cleanup.

Flow: orient -> decide what matters -> update the repo -> commit/push -> report the handoff.

## 1. Orient

1. Find the repo root with `git rev-parse --show-toplevel`; if that fails, use the current directory and ask before `git init`.
1. Check `git status --short` and `git diff --stat`. Treat existing dirty files as possible in-flight work, not as noise to overwrite.
1. Discover existing state docs before creating new ones: `AGENTS.md`, `CLAUDE.md`, `PROGRESS.md`, `STATUS.md`, `TODO.md`, `NOTES.md`, `README.md`, `docs/`, ADRs, issue trackers, or project-specific equivalents.
1. Read only the files needed to understand where state belongs.

## 2. Decide What To Preserve

Use judgment. Preserve what helps someone resume; skip ceremonial edits.

Write down project-scoped state such as:

- Current goal, completed work, in-flight work, blockers, and the next useful step.
- Decisions, constraints, conventions, architecture notes, and commands that matter later.
- Test/build/verification results and known failures.
- Files touched or important uncommitted changes, when that context is not obvious from the diff.
- Open questions, follow-ups, and owner/action context.
- Project-specific memory from the running agent that is not already in the repo.

Do not write user-personal or global facts into the repo unless the user explicitly says they belong there. If there is no meaningful new project state and no relevant working-tree change, say so and skip the commit.

## 3. Update The Right Files

Prefer existing project files over new files. Create a new state file only when there is useful state and no existing place for it.

Common routing:

- `AGENTS.md` or equivalent: durable project conventions and how to work in the repo.
- `PROGRESS.md`, `STATUS.md`, or `NOTES.md`: current handoff state, recent progress, decisions, and blockers.
- `TODO.md` or issue tracker references: open actions and unresolved questions.
- `README.md`: user-facing usage, install, or project description updates.
- `docs/` or ADRs: design rationale future contributors need.

Keep the handoff concise, dated when useful, and AI-agnostic. Preserve unrelated sections. Do not create a progress file just to say nothing happened.

## 4. Commit And Push

Invoking `/flush` authorizes committing and pushing the state needed for handoff.

1. If the directory is not a git repo, ask before `git init`.
1. Stage only files that should travel to the next machine. Do not use `git add -A` blindly.
1. If both product changes and handoff docs exist, decide whether one commit or separate commits is clearer.
1. Use a direct message such as `docs: record handoff state`, `checkpoint: save current project state`, or a project-specific summary.
1. Push:
   - If a remote exists, run `git push`. If upstream is missing, push with `-u origin <branch>`.
   - If no remote exists, ask before `gh repo create <name> --source=. --push`, including whether it should be private or public.
   - If auth fails, report the error and stop trying to fix credentials.

Push failures do not erase the handoff work; report what remains local.

## 5. Final Report

Tell the user:

- Which files were updated and why.
- What was committed, including commit SHA.
- Whether push succeeded or why it did not.
- Any remaining uncommitted changes.
- The shortest path for the next session: which file to read first and what the next action is.
