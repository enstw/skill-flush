---
name: recommend
description: Pause the current trajectory, step back, and surface recommendations or refactors before continuing. Use when the user asks for "recommendation or refactor", "step back", "are we on track", "is this getting messy", or any other direction check. Also self-trigger when you (the running agent) sense the work is drifting from its original goal in the current session - signals include back-and-forth edits, scope creep, naming churn, half-finished implementations, growing surface area without simplification, or accumulated decisions that could be reconciled into one cleaner pattern. Better to surface drift early than ship it.
user-invocable: true
allowed-tools:
  - Read
  - Edit
  - Write
  - Bash(pwd)
  - Bash(ls *)
  - Bash(find *)
  - Bash(grep *)
  - Bash(git status *)
  - Bash(git diff *)
  - Bash(git log *)
  - Bash(git show *)
---

# /recommend - reconsider direction, recommend or refactor

Pause the current execution trajectory. Step back, look at where the work has landed, surface recommendations for course correction, and offer to refactor. This is a check on *direction*, not a code review.

## When to invoke

Direct invocation: user runs `/recommend`, or asks for "recommendation or refactor", "step back", "are we on track", "is this getting messy", or any equivalent direction check.

Self-invocation when you sense drift in the current session:

- Back-and-forth edits in opposite directions on the same code.
- Scope creep beyond the original ask.
- Naming inconsistencies emerging across files (folder name vs. frontmatter name vs. heading).
- Half-finished implementations, dead code, or branches with no consumer.
- Surface-area growth without proportional simplification.
- Multiple decisions that could be reconciled into one cleaner pattern.

If none of these apply and the trajectory is healthy, say so briefly and skip the recommendation list. Do not invent drift just to justify firing the skill.

## Flow

1. **Re-read the goal.** What was the original ask in this session, and what is the current trajectory? If the goal has implicitly shifted, name the shift.
1. **Diff against trajectory.** Look at `git status`, `git diff`, recent commits in the session, and edits made in the conversation. Identify changes that drift from the goal or contradict each other.
1. **List recommendations**, ordered by payoff. Each one is concrete: file, line, what to change, why.
   - Prefer 1-5 items. If the list is longer, the recommendations are not yet sharp enough - keep cutting until each one earns its place.
   - Mark each as **must-do** (a real bug or contradiction), **should-do** (clear improvement), or **optional** (taste call).
   - Lead with user-visible impact, not implementation detail.
1. **Ask the user which to apply.** Do not refactor without confirmation. If the user picks all/some, do them; if they pick none, stop.
1. **Refactor in one commit per recommendation, or one bundled commit if items are tightly related.** Match the repo's existing commit style.
1. **Report what changed.** Files touched, commit SHAs, anything left undone.

## Guarantees

- **No silent rewrites.** The skill always lists recommendations and asks before changing code.
- **No invented drift.** If the trajectory is healthy, the skill says so and stops.
- **Direction first, polish second.** Surface architecture, scope, and naming issues before nitpicks. Lint and style fixes belong in a different skill.
- **Bounded scope.** Recommendations cover work done in the current session, not the whole repo.
- **Self-triggering is opt-in to *suggesting*, not to *acting*.** Even when the agent self-triggers on sensed drift, it stops at the recommendation list and asks before refactoring.
