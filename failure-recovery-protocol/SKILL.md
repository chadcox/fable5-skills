---
name: failure-recovery-protocol
description: Structured hypothesis-driven recovery whenever a command, build, test, or tool call fails. Use this skill on EVERY non-trivial failure — failing tests, compile errors, tool errors, unexpected output, environment problems — before attempting a second try. Trigger especially when tempted to re-run the same command hoping for a different result, or after two consecutive failed fix attempts on the same problem.
---

# Failure Recovery Protocol

## Why this exists

There are two classic failure spirals: the **retry loop** (re-running variations of the same failing thing, burning an hour and half the context window) and the **premature surrender** ("I'm stuck" after two shallow attempts). Skilled operators recover from their own failures because they treat each failure as *information* and each retry as an *experiment*. This protocol enforces that.

Iron rule: **never run the same failing command twice without a changed hypothesis.** If nothing changed, the outcome won't either.

Escalation trigger: if the same command, test, build, tool call, or attempted fix fails twice, stop ordinary execution and use this protocol even if the failure looked small at first.

## The protocol

### 1. Capture the failure completely

Read the **entire** error output — full stack trace, stderr, exit code — not just the last line. Copy the decisive lines into `.codex/STATE.md` (see `working-memory-ledger`) under Gotchas if this looks non-trivial. Half the time, the real error is 30 lines above the one that caught your eye.

### 2. Classify

| Class | Signature | Typical response |
|---|---|---|
| **Environment** | missing dep, wrong version, PATH, permissions, network | Fix env; don't touch code |
| **Wrong assumption** | API doesn't work how you thought, schema differs, flag doesn't exist | Go read the actual source/docs before editing |
| **Logic bug** | your code does the wrong thing | Normal debugging |
| **Flaky/external** | timeout, rate limit, intermittent | Retry ONCE deliberately; if it recurs, treat as real |
| **Pre-existing** | fails on an unmodified baseline too | Check `git status --short` first; verify on a clean worktree or temporary worktree when safe; report, don't own it silently |

The "pre-existing" check matters: before assuming you broke it, confirm it worked before you arrived. Do not stash or hide unknown user changes just to run this check; ask or use a separate worktree when ownership is unclear.

### 3. Hypothesize, then test the hypothesis

State (to yourself, in one sentence, ideally in `.codex/STATE.md`): *"I believe X failed because Y; if I'm right, then Z should be observable."*

Then **test Z — the cheapest observable prediction — not the full fix.** Examples:

- Hypothesis: import fails due to version skew → check `pip show <pkg>` before rewriting the import.
- Hypothesis: WinRM call fails due to auth, not code → test with a bare `Test-WSMan` before touching the script.
- Hypothesis: test fails from stale fixture → run the single test in isolation before "fixing" the code it tests.

If the observation contradicts the hypothesis, form a new one. Do **not** apply a fix for a hypothesis you haven't confirmed — unconfirmed fixes are how one bug becomes three.

### 4. Fix minimally, verify, record

Apply the smallest change that addresses the *confirmed* cause. Re-run the original failing command. On success, run `self-verification-loop` on the fix, and record the gotcha in `.codex/STATE.md` so it's never re-derived.

### 5. Escalation budget

Cap yourself at **3 distinct hypotheses** (not 3 retries — 3 genuinely different theories, each tested). After that, stop and escalate to the user with a structured summary:

```
## Stuck: <one-line problem statement>
- Symptom: <exact error, trimmed>
- Tried:
  1. Hypothesis A (env) — disproven by <observation>
  2. Hypothesis B (schema) — partially confirmed but fix regressed <thing>
  3. Hypothesis C (upstream bug) — plausible; evidence: <link/lines>
- Current best theory + what I'd need to confirm it (access, info, a decision)
```

A structured escalation after 3 real attempts is a *good* outcome. It preserves everything learned and lets the user unblock you in one message. Grinding silently for an hour and then escalating with "it doesn't work" wastes both the hour and the learning.

## Anti-patterns

- Retry-with-tweaks: changing something random each attempt so it *feels* like progress. If you can't state the hypothesis, it's a guess.
- Shotgun fixes: applying three candidate fixes at once. You won't know which one worked, and the other two are now landmines.
- Suppressing instead of fixing: `|| true`, broad `except:`, skipping the failing test. These convert loud failures into silent ones.
- Forgetting the fix: solving a gnarly environment issue and not recording it, guaranteeing a repeat performance next session.
