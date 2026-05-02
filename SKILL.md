---
name: flush
description: End a session safely. Migrate this project's knowledge out of agent-private memory and into the repo's living state docs (AGENTS.md, PROGRESS.md, TODO.md, etc.), commit and push, then wipe this project's agent memory. AI-agnostic output — the docs work for any future agent or human reader. Use when wrapping up, switching tools, or wanting a clean slate.
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash(pwd)
  - Bash(git rev-parse --show-toplevel)
  - Bash(git status --short)
  - Bash(git diff --stat)
  - Bash(git init)
  - Bash(git add *)
  - Bash(git commit *)
  - Bash(git push)
  - Bash(git push *)
  - Bash(git remote *)
  - Bash(gh repo create *)
  - Bash(ls *)
---

# /flush — This project's memory → repo, then wipe

**Principle (AI-agnostic):** the repo owns project knowledge. Agent-private memory is a cache. Before wiping the cache for this project, anything project-specific has to land in the repo.

This applies to whichever agent is running this skill. The steps below describe the work in agent-neutral terms — you adapt the concrete paths for your own memory store.

Flow: migrate memory → record session state → commit/push → wipe.

---

## Step 0 — Locate the repo

1. Run `git rev-parse --show-toplevel`. Use that as the root; fall back to cwd if it errors (not a git repo).
1. If it's a git repo, run `git status --short` and `git diff --stat` — uncommitted work is part of state. If not, skip.

## Step 1 — Migrate this project's memory to the repo

Read your own per-project memory for this project. Sort each entry by **type / scope**:

| Memory content | Where it goes |
|---|---|
| Project conventions, architecture, how-to-work-on-this-repo | `AGENTS.md` (or `CLAUDE.md` if the repo uses that) |
| Current goal, in-flight work, recent decisions | `PROGRESS.md` |
| Open action items, undecided questions | `TODO.md` |
| Design rationale future contributors need | `docs/` or an ADR |
| Facts about the **user themselves** (not tied to this project) | **Keep in agent memory.** Do not dump user-personal facts into the repo. |
| Pointers to external systems (Linear, Slack, dashboards) | `AGENTS.md` references section, only if useful for future contributors |

**Rules:**

- **Distinguish user-scope from project-scope memories.** User-scope (preferences, role, who-they-are) stays in agent memory. Project-scope migrates to the repo. If unsure, ask.
- **Discover, then read, then write.** Look at the root for existing state docs (`AGENTS.md`, `CLAUDE.md`, `PROGRESS.md`, `STATUS.md`, `TODO.md`, `NOTES.md`) and any agent-specific files (`.cursorrules`, `.aider.md`, etc.). Read before writing; match what's there and don't create parallel files. Preserve sections you have no update for.
- **Stay AI-agnostic in the docs.** Write for any reader — agent or human. No tool-specific paths or instructions in the body.
- **De-duplicate.** If the memory entry is already in code or in an existing doc, drop it; don't restate.

If your per-project memory is empty or contains only user-scope entries, skip to Step 2.

## Step 2 — Add this session's state

Reflect on the current conversation. Anything new that isn't in memory or in the docs yet:

- New decisions, conventions, or constraints surfaced this session → `AGENTS.md`.
- Current goal, what's done / in flight / blocked, files touched, uncommitted changes → `PROGRESS.md`.
- New open questions or action items → `TODO.md`.

Same routing rules as Step 1. Match each doc's existing list and heading style.

If the project has no state docs at all and there's meaningful content to record, ask the user which to create (default: `PROGRESS.md` + `TODO.md`). Don't decide unilaterally.

If nothing meaningful surfaced this session, say so and proceed to Step 3.

## Step 3 — Commit and push

Invoking `/flush` authorizes commit + push of the doc changes from Steps 1–2. This skill targets solo repos — no PR step.

1. If not a git repo, ask whether to `git init` so commit/push can happen. If declined, skip this step.
1. If nothing changed in Steps 1–2, skip.
1. `git add` only the docs you updated. Don't `git add -A` — there may be unrelated working-tree changes.
1. `git commit -m "<short message describing the doc update>"` (e.g. `docs: flush — migrate memory + record session state`).
1. **Push.** Check for a remote with `git remote -v`:
   - **Remote exists:** `git push`. If it fails on missing upstream, run `git push -u origin <branch>`. If it fails on auth, report the error verbatim and stop — do not try to fix auth yourself. (`gh auth` only covers HTTPS pushes to github.com after `gh auth setup-git`; SSH remotes use SSH keys, which gh doesn't manage. Let the user diagnose.)
   - **No remote:** ask the user whether to publish with `gh repo create <name> --source=. --push` and what visibility (`--private` or `--public`). Repo creation is publicly visible and not easily reversed — **ask first**. If they decline, skip push.
1. Push failures do not block Step 4. Report and continue.

## Step 4 — Confirm and wipe (this project only)

The wipe is mandatory — that's the whole point. Project memory must not survive past the session.

1. Wipe your own per-project memory for this project. **Scope: this project only** — do not touch other projects' memory or any user-scope/global memory.
1. Show the user what you're about to delete and the exact command, then ask for explicit confirmation. **Do not wipe without a clear yes.**
1. On confirmation, run it. Report what was cleared.

## Closing message

Tell the user:

- Which docs were updated, one line each.
- How many memory entries were migrated, from where.
- The commit SHA and whether the push succeeded (or why it was skipped).
- (If wiped) that this project's agent memory is gone — the repo is now the only record. The next session, in any agent, can read the state docs to get oriented.

---

## Notes

- The repo is the source of truth. Memory is convenience.
- User-personal memories (preferences, role) stay in agent memory — those don't belong in a project repo.
- Wipe scope is **this project's memory only** — other projects' memory is untouched. If the user wants a global wipe, they'll ask for it explicitly.
- Invoking `/flush` is itself the user's authorization to commit and push the doc updates. No PR step — this skill is for solo repos.
