---
name: task-decomposition-planner
description: Produce a written, persisted execution plan before starting gnarly, long-running, ambiguous, or high-blast-radius coding work. Use this skill when effort-calibration classifies a task as gnarly, the work is expected to exceed ~30 minutes, a migration/broad refactor is involved, requirements are ambiguous, or durable state would materially prevent drift. For standard 1-3 file work, prefer an inline checklist unless the task expands. Trigger BEFORE writing code once those conditions are met.
---

# Task Decomposition Planner

## Why this exists

The dominant failure mode on long tasks is drift: the model starts strong, then loses the thread mid-execution — requirements get silently dropped, subtasks are half-finished, and effort goes to whatever file is currently open rather than what the task needs. A persisted plan that is re-read at every phase boundary is cheap insurance against all of it.

The plan is not for the user (though they can read it). It is your own working document. Its value comes from being **written down and re-read**, not from existing in your head.

## Workflow

### 1. Classify first

Use `effort-calibration` first. If the task is trivial, execute directly. If it is standard, use an inline checklist unless it starts sprawling. Create a persisted plan only when the task is gnarly, long-running, ambiguous, high blast-radius, or likely to survive context compaction. When in doubt about those signals, plan — a small plan is cheaper than losing a complex task.

### 2. Write .codex/PLAN.md

Create `.codex/PLAN.md` by default so planning artifacts do not pollute the project root. If the repo already has a local convention for agent artifacts, use that instead. Use this structure:

```markdown
# Task: <one-line restatement of the user's actual request>

## Requirements (verbatim-ish from the user)
1. R1: ...
2. R2: ...
3. R3 (implicit): ...   <- things the user clearly expects but didn't state

## Success criteria
- How we'll know each requirement is met (tests pass, command output, behavior observed)

## Subtasks
- [ ] S1: <subtask> — files: <likely files> — depends on: none
- [ ] S2: <subtask> — files: <...> — depends on: S1
- [ ] S3: ...

## Execution order
S1 -> S2 -> (S3 || S4) -> S5

## Risks / open questions
- Anything ambiguous to confirm with the user BEFORE starting, not after
```

Rules for good subtasks:

- Each subtask should be independently verifiable — you can run something and know it worked.
- Each should end in a stable state (compiles, tests pass) so it can pair with `long-horizon-checkpointing`.
- Explicitly note dependencies. Parallel-safe subtasks matter if `parallel-work-splitter` is in play.
- Include the boring subtasks: "update call sites", "update docs", "run full suite". These are the ones that get dropped.

### 3. Surface ambiguity now

If any requirement can be read two ways and the readings lead to different implementations, ask the user **before** executing. One clarifying question up front beats a rewrite later. If the ambiguity is minor, state your assumption in `.codex/PLAN.md` and proceed.

### 4. Execute against the plan

- Work one subtask at a time. Check it off in `.codex/PLAN.md` only after it is verified (see `self-verification-loop`).
- **At every subtask boundary, re-read `.codex/PLAN.md`.** This is the anti-drift mechanism. Ask: am I still doing what the plan says? Has anything I learned invalidated a later subtask?
- If reality diverges from the plan (it will), **update the plan file** — add/remove/reorder subtasks and note why. A stale plan is worse than no plan.

### 5. Close out against the plan

Before final handoff, walk the Requirements list one item at a time and confirm each maps to completed, verified subtasks. Hand this list to `self-verification-loop` step 4. Any requirement that didn't survive must be explicitly reported, never silently dropped.

## Example

User: *"Migrate our alert enrichment from the old ServiceNow SIR webhook to TheHive 5 API, and make sure the Slack notifications still work."*

Bad response: start editing the webhook handler immediately.

Good response: `.codex/PLAN.md` with requirements R1 (TheHive API integration), R2 (Slack notifications preserved — regression requirement, needs its own verification), R3 implicit (old SIR path removed or feature-flagged?  ← ask the user), subtasks covering API client, payload mapping, error handling, Slack regression test, and cleanup.

## Anti-patterns

- Writing the plan and never re-reading it. The re-read at boundaries **is** the skill.
- Plans with vague subtasks like "implement the feature". If a subtask can't fail verification, it's not a subtask.
- Treating the plan as immutable. Update it when you learn things.
- Planning in chat prose only. It must be a file — files survive context compaction; chat scrollback effectively doesn't.
