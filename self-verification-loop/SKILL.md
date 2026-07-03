---
name: self-verification-loop
description: Mandatory self-verification workflow before declaring any coding task complete. Use this skill on EVERY task that modifies code, config, or infrastructure — bug fixes, features, refactors, script changes, IaC, CI edits — even small ones. Trigger whenever you are about to say "done", "fixed", "that should work", or hand results back to the user. Do not skip it because the change "looks trivial"; trivial-looking changes are where unverified regressions hide.
---

# Self-Verification Loop

## Why this exists

The most common failure mode on coding tasks is declaring victory after the first pass: the code compiles in your head, so you report success. Frontier-model behavior differs mainly in discipline, not intelligence — verify output, iterate on failures, and only then respond. This skill makes that loop mandatory.

Core rule: **"Done" is a claim that requires evidence.** If you cannot point to a concrete verification artifact (test output, command output, a diff you re-read against the requirement), you are not done — you are hopeful.

## The loop

Run this after every logically complete change, before reporting completion:

### 1. Re-read the original requirement

Go back to the user's actual request (and the plan file, if one exists from `task-decomposition-planner`). List each requirement explicitly. Do not work from your memory of the request — memory drifts over long sessions.

### 2. Verify mechanically

In order of preference:

1. **Run the tests.** Existing test suite first (scoped to affected modules if the suite is huge, full suite before final handoff). If no tests cover the change, write at least one that exercises the new behavior and one that exercises the failure/edge case.
2. **Run the code.** Execute the actual entry point with realistic input. For scripts: run them. For servers: start them and hit the endpoint. For PowerShell: actually invoke the function, don't just define it.
3. **Static checks.** Linter, type checker, `terraform validate`, `python -m py_compile`, `pwsh -NoProfile -Command "Set-StrictMode -Version Latest; . ./script.ps1"` — whatever the ecosystem offers.
4. **Read the full diff.** `git diff` end to end. Look for: debug statements left in, half-renamed identifiers, TODO stubs, changes to files you didn't intend to touch.

### 3. Actually read the failures

When something fails, read the entire error output before acting. Do not pattern-match on the first line and guess. Extract: what failed, where, and what the runtime actually saw vs. what you expected. If a failure occurs, hand off to `failure-recovery-protocol` rather than blind-retrying.

### 4. Diff against requirements

For each requirement from step 1, mark it: **verified** (evidence exists), **implemented-unverified** (say so explicitly), or **not done**. Anything not verified must be disclosed in your final report — never let the user discover it.

### 5. Adversarial pass

Before handoff, ask: *"What would a senior engineer flag in this diff?"* Check specifically:

- Edge cases: empty input, nulls, unicode, huge input, concurrent access
- Error paths: what happens when the network call fails, the file is missing, the API returns 500
- Security: injection points, secrets in code, permissions widened
- Consistency: does the change follow the codebase's existing conventions, or did you introduce a second pattern?

Fix what you find, then re-run step 2. Loop until a full pass is clean.

## Reporting format

End every completion report with a verification summary:

```
## Verification
- [x] `pytest tests/auth/ -x` — 42 passed
- [x] Manual run: `./onboard.ps1 -Hostname TEST01 -WhatIf` — output correct
- [x] Full diff reviewed — no unintended files touched
- [ ] NOT verified: behavior under concurrent enrollment (no test harness available) — flagged for user
```

An honest "not verified, here's why" is a success. A confident "done" that fails on first use is the failure this skill exists to prevent.

## Anti-patterns

- **Compiling ≠ working.** A clean build verifies syntax, not behavior.
- **"The logic is straightforward" ≠ evidence.** Run it.
- **Testing only the happy path.** At minimum, exercise one failure path.
- **Skipping verification because the session is long and you want to finish.** Long sessions are exactly when drift has accumulated and verification matters most.
