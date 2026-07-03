---
name: long-horizon-checkpointing
description: Structure long coding sessions into resumable git checkpoints so work survives interruption, context compaction, or session death. Use this skill on any task expected to run more than ~30 minutes or span multiple sittings — migrations, large refactors, multi-subtask builds — and ALWAYS when resuming previous work. Trigger at the start of long tasks, at every stable state, and before any risky/sweeping change.
---

# Long-Horizon Checkpointing

## Why this exists

Long agentic runs die: context compacts, sessions time out, the user closes the laptop. Without checkpoints, a died-at-80% run resumes at 0% — or worse, resumes at a *guessed* 80% and corrupts good work. Multi-hour stamina is mostly the ability to persist and re-load state. Git plus a disciplined ledger gives you that.

Definition: a **checkpoint** = a git commit at a *stable state* + an updated `STATE.md` + PLAN.md progress marks. All three, together. A commit without the ledger update is a snapshot nobody can interpret.

## Workflow

### 1. Establish the safety net at task start

```bash
git status                      # never start with a dirty tree you don't understand
git checkout -b task/<name>     # work on a branch; main stays clean
git log --oneline -5            # know your baseline
```

If the tree is dirty with someone else's changes, stop and ask the user before proceeding.

### 2. Checkpoint at every stable state

A stable state = compiles, relevant tests pass, and one coherent unit of the plan is done (typically one subtask from `task-decomposition-planner`). At each one:

```bash
git add -A && git commit -m "S3: map TheHive payload fields (14/22) — enums pending, see STATE.md"
```

Commit message rules:

- Reference the subtask ID from PLAN.md.
- State what is DONE and what is explicitly NOT done.
- Meaningful messages only — a log of `wip`, `wip2`, `fix` is unresumable.

Then update STATE.md ("Currently in progress" / "Next actions") and tick PLAN.md. **The commit and the ledger update are one atomic ritual.**

### 3. Checkpoint before risk

Before any sweeping operation — codemod, mass rename, dependency upgrade, destructive migration script — commit first, even mid-subtask. Label it: `pre-risk: before sed rename of EnrichedAlert -> Alert`. This makes `git reset --hard` a one-command undo instead of an archaeology project.

### 4. Never checkpoint garbage

Do not commit known-broken states as checkpoints (exception: `pre-risk` snapshots, clearly labeled). If interrupted while broken, use `git stash` with a message, and record the stash's existence in STATE.md — an unrecorded stash is a lost stash.

### 5. Cold-resume procedure

When resuming any prior session, reconstruct from artifacts, never from memory:

```bash
git log --oneline -10   # what was the last stable state?
git status              # anything uncommitted/stashed?
git stash list
```

Then read STATE.md and PLAN.md, reconcile all three, and state (briefly, to the user or in the ledger): *"Resuming at S4; S1–S3 verified per commits abc123..def456."* If artifacts disagree — uncommitted changes not described in the ledger — investigate and reconcile **before** writing new code. When in doubt about uncommitted work, stash it rather than building on top of it.

### 6. Landing the work

Before handoff: run the full verification pass (`self-verification-loop`), then review the branch history. Offer the user either the branch as-is (honest history) or a squashed/tidied version — their call, their repo conventions. Never force-push over anything you didn't create this session.

## Anti-patterns

- One giant commit at the end. That's not checkpointing; that's gambling the whole session on nothing going wrong.
- Commits without ledger updates. Six months of `git log` cannot tell you what "Next actions" were.
- Committing secrets or scratch files because `git add -A` was reflexive. Glance at `git status` first; maintain .gitignore.
- Building on top of mystery uncommitted changes found at resume time.
