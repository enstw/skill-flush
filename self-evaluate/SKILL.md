---
name: self-evaluate
description: Estimate how many PDCA (plan-do-check-act) loops remain before the work is finished. The number is cost-driven - each loop costs time, tokens, and attention - and is invokable in any phase: pre-implementation, mid-implementation, post-test-failure. The agent investigates (reads code, smoke-tests, checks env, web-searches) to reduce uncertainty before estimating, so the number is grounded in evidence rather than guessed. Use when the user asks "how many more loops", "how close are we", "self-evaluate", "are we close to done", or wants a cost-driven estimate of remaining work.
user-invocable: true
allowed-tools:
  - Read
  - Bash
  - WebFetch
  - WebSearch
---

# /self-evaluate - PDCA loops remaining

Estimate how many plan-do-check-act loops the running agent (with its current tools and environment) needs to take this work from where it is now to done.

This is cost-driven: each loop costs compute, tokens, and human attention. The estimate tells the user how much budget to expect, whether to keep iterating or replan, or whether to switch tools entirely. It is calibration, not a commitment.

The estimate must be **earned through investigation**. Guessing without checking the code, environment, current diff, or unknowns is the failure mode this skill exists to prevent.

The skill is **phase-agnostic**. It can be invoked when there is only a plan, mid-implementation with code in flight, after a failed test run, or at any point in between. The flow is the same; only the artifact under evaluation changes.

## Flow

1. **Identify what is being evaluated.** The named artifact (path, PR number, "the plan above", "the work in flight") or the most recent plan/recommendation/diff in the session. Note the phase: pre-implementation, mid-implementation, post-test-failure, etc. If nothing evaluable is in scope, ask. If the artifact is too vague even after asking, refuse - refusal is a valid output.

1. **Investigate to reduce uncertainty.** Time-box proportional to the work's blast radius: a one-file refactor needs minutes, a multi-system migration may justify a longer probe. Pick the cheapest checks that move the estimate most. The investigation itself is meta-work, not a loop - it is the preparation that makes the count more accurate.

   - **Code grounding.** Open the files the work touches. Verify symbols, line numbers, function signatures, schema fields. References to things that don't exist mean more loops.
   - **Current state.** If mid-implementation, read the current diff. What's done, what's broken, what's in flight? If post-test-failure, read the actual error output and try a minimal repro.
   - **Environment assessment.** Are the tools the work assumes installed and at the right versions? Are credentials, network, permissions in place? Can this agent actually run the commands the work requires? Missing tools mean more loops, possibly unbounded.
   - **Smoke tests.** Run the build, run the existing tests, run a CLI the work depends on - does the starting state even work? You cannot estimate fix loops on top of a broken baseline.
   - **Web search.** Unfamiliar library API shape, version-specific behavior, error messages, recent breaking changes. Especially worth it for any third-party dependency the work leans on.
   - **Scratch experiments.** When the work hinges on a single uncertain mechanism (regex behavior, API edge case, library quirk), run a tiny throwaway probe rather than guess.

   Stop investigating when the next likely check would not move the estimate by more than one loop.

1. **Estimate** using the anchors below. Use a range when the uncertainty is real - false precision helps no one.

1. **Report** in the output shape below.

## Anchors

| Loops remaining | Meaning |
|---|---|
| 0 | Done. Tests pass now. Reserved for confirmed completion. |
| 1 | One round: implement (or polish) + verify, ship. |
| 2-4 | Normal iteration on a grounded plan. Edge cases, test fixtures, library-edge behavior. |
| 5-10 | Multiple unverified assumptions. Each loop will reveal more. |
| 10+ | Plan needs rework before implementation; replan rather than push through. |
| Unbounded | Fundamentally stuck. Direction is wrong, the environment can't support it, or scope is unclear. Stop and rethink. |

Anchor rules:

1. **0 loops** requires tests have actually passed - not "the plan looks right".
1. **1 loop** requires the plan is verified end-to-end against code AND a working build environment.
1. **2-4 loops** requires the plan's claims have been verified against the actual code.
1. **10+ or Unbounded** should come with a recommendation to replan rather than execute. Don't bury that under a number.

## Output shape

1. **Loops remaining: N** (or **N-M**, or **Unbounded**). One line.
1. **Where they'll land.** Bulleted breakdown - "~1 loop on build setup, 2-3 on integration tests, 1 on polish". This *is* the body of the estimate; the number must be justified by this list.
1. **What could push it higher.** Concrete risks. Each cites a file:line, a missing tool, a known unknown, or an assumption the time budget did not cover. If you cannot name at least one, the estimate is too low.
1. **What pulls it lower.** Concrete grounding from the investigation. Each cites a verified file, a passing smoke test, a confirmed tool version, or a settled search result. If you cannot name at least one, the estimate is too high.
1. **Investigations performed.** One-line bullets: what was checked, what it confirmed or invalidated. Makes the estimate auditable. Skip only when the work was so small that no investigation was warranted - and say that explicitly.

## Hard rules

1. **Investigate before estimating.** A count with no investigation step is unsupported. If the time budget truly allowed nothing, say "insufficient investigation - tentative N loops" rather than a clean number.
1. **No estimate without evidence.** Every supporting claim must cite a file:line, command output, search hit, or smoke-test result.
1. **No invented confidence.** If a key assumption could not be verified, list it under "what could push it higher" rather than estimating around it.
1. **No estimate drift from pushback.** If the previous estimate this session was 5 loops and nothing material changed, the new estimate is still 5. Pushback is not new evidence; new investigation is.
1. **Refusal is valid.** If the artifact is too vague to estimate even after investigation, return "insufficient detail to estimate" with a concrete list of what is missing.
1. **Unbounded is valid.** Don't force a number when the work is fundamentally blocked. "Unbounded - replan first" is a real answer.

## Second opinion (optional)

When the user asks for a second opinion, spawn a fresh-context reviewer - a subagent, a separate session, or a different tool - and give it the same rubric and the same artifact. Report both estimates side by side. Disagreements of more than 2x (e.g. one says 2, the other says 5+) are worth surfacing as a discussion, not auto-reconciling. This skill body is portable enough that the user can run the same rubric in a different tool by hand.
