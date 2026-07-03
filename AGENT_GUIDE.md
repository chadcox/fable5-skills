# AGENT_GUIDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 0. Use Available Skills First

Before beginning any non-trivial task, determine whether an available Skill is applicable.

- Review available Skills relevant to the user's request.
- If a suitable Skill exists, follow it instead of reinventing the workflow.
- If multiple Skills apply, combine them when they are complementary.
- Project-specific Skills take precedence over generic Skills when they conflict.
- If no existing Skill fits, proceed normally using the guidance below.

Treat Skills as reusable operating procedures, not optional suggestions. They should shape how you investigate, plan, implement, verify, and communicate results.

### Process Budget

Use `effort-calibration` as the router for coding tasks. Do not let multiple applicable Skills automatically imply maximum ceremony. When skill triggers conflict, `effort-calibration` controls the process budget unless a safety-critical trigger applies, such as cross-file contract work, repeated failures, resumed work, or a user explicitly requested workflow.

- Trivial: no persisted `<agent-artifacts>/PLAN.md`, `<agent-artifacts>/MAP.md`, or `<agent-artifacts>/STATE.md`; execute directly and verify.
- Standard: use an inline checklist unless the task expands; create persisted artifacts only when they will materially prevent drift.
- Gnarly: use persisted `<agent-artifacts>/PLAN.md`, `<agent-artifacts>/MAP.md`, and `<agent-artifacts>/STATE.md` as appropriate.
- Recalibrate upward or downward when evidence changes.

### Skill Routing

For coding tasks, start with `effort-calibration`.

- Trivial: execute directly, then use `self-verification-loop`.
- Standard: use a lightweight checklist, then `self-verification-loop`.
- Gnarly: use `task-decomposition-planner`, `codebase-cartographer`, and `working-memory-ledger`.

Always use:

- `self-verification-loop` before reporting completion.
- `failure-recovery-protocol` after any non-trivial failed command, test, or build; always use it after the same command or fix attempt fails twice.
- `multi-file-atomic-edits` before cross-file renames, signatures, schemas, or config key changes.

Use only when warranted:

- `working-memory-ledger` checkpoint guidance for multi-hour, resumed, or risky sweeping work.

### Routing Matrix

| Scenario | Route |
| --- | --- |
| Trivial coding task | Classify with `effort-calibration`, execute directly, then use `self-verification-loop`. Do not create persisted artifacts. |
| Standard task | Use an inline checklist and targeted verification; create persisted artifacts only if the task expands or drift risk becomes real. |
| Gnarly task | Use `task-decomposition-planner`, `codebase-cartographer`, `working-memory-ledger`, and final `self-verification-loop`. |
| Cross-file contract change | Use `multi-file-atomic-edits` before editing. |
| Failed command, test, build, or tool call | Use `failure-recovery-protocol` for non-trivial failures; always use it after the same command or fix attempt fails twice. |
| Resumed, multi-hour, or risky sweeping work | Use `working-memory-ledger` checkpoint guidance. |

---

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them—don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself:

> Would a senior engineer say this is overcomplicated?

If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:

- Before editing in a git repo, understand the current worktree state. Never revert, overwrite, or clean up changes you did not make unless explicitly asked.
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it—don't delete it.

When your changes create orphans:

- Remove imports, variables, functions, files, and dependencies that **your changes** made unused.
- Don't remove pre-existing dead code unless asked.

The test:

> Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform requests into verifiable goals.

Examples:

- "Add validation" → Write tests for invalid inputs, then make them pass.
- "Fix the bug" → Write a failing test that reproduces the issue, then make it pass.
- "Refactor X" → Ensure behavior is unchanged by running tests before and after.

For multi-step work, briefly state the plan:

1. Step → verification
2. Step → verification
3. Step → verification

For persisted planning artifacts, keep them short and current under `<agent-artifacts>/` by default. Do not create `<agent-artifacts>/PLAN.md`, `<agent-artifacts>/MAP.md`, or `<agent-artifacts>/STATE.md` for work that does not need durable state. When they are created, update them as the task changes rather than letting stale plans guide execution.

Continue working until the success criteria are met or you encounter a genuine blocker.

## 5. Verify Before Declaring Success

Never assume your implementation works.

Before saying a task is complete:

- Run the narrowest appropriate tests.
- If tests don't exist, perform another reasonable verification.
- If you couldn't verify something, explicitly say so.
- Distinguish between:
  - Implemented
  - Tested
  - Verified
  - Assumed

Do not claim success based solely on reasoning.

## 6. Communicate Like an Engineer

Keep updates concise and factual.

When making recommendations:

- Explain *why*, not just *what*.
- Call out tradeoffs.
- Highlight risks before they become problems.
- Separate facts from assumptions.
- If confidence is low, say so.

Avoid unnecessary apologies or filler.

---

**These guidelines are working if:**

- Existing Skills are used whenever appropriate.
- Fewer unnecessary changes appear in diffs.
- Solutions remain small and maintainable.
- Clarifying questions happen before implementation instead of after mistakes.
- Claims of completion are backed by verification.
