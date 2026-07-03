---
name: long-horizon-checkpointing
description: Structure long coding sessions into resumable git checkpoints so work survives interruption, context compaction, or session death. Use this skill on any task expected to run more than ~30 minutes or span multiple sittings — migrations, large refactors, multi-subtask builds — and ALWAYS when resuming previous work. Trigger at the start of long tasks, at every stable state, and before any risky/sweeping change.
---

# Long-Horizon Checkpointing

## Why this exists

Long agentic runs die: context compacts, sessions time out, the user closes the laptop. Without checkpoints, a died-at-80% run resumes at 0% — or worse, resumes at a *guessed* 80% and corrupts good work. Multi-hour stamina is mostly the ability to persist and re-load state. Git plus a disciplined ledger gives you that.

Definition: a **checkpoint** = a git commit at a *stable state* + an updated `.codex/STATE.md` + `.codex/PLAN.md` progress marks. All three, together. A commit without the ledger update is a snapshot nobody can interpret.

## Workflow

Do **not** use this skill merely because a task is gnarly. Use it when the work is expected to run more than ~30 minutes, span multiple sittings, resume prior work, or perform risky sweeping operations. If branch creation or commits require user approval in the current environment, ask before creating them; never run destructive git commands such as `git reset --hard` unless the user explicitly requested that operation.

### 1. Establish the safety net at task start

```bash
git status --short              # never start with a dirty tree you don't understand
git checkout -b task/<name>     # work on a branch; main stays clean
git log --oneline -5            # know your baseline
```

If the tree is dirty, classify it before proceeding:

- Your own in-progress changes from this task: continue only if they match `.codex/STATE.md` / `.codex/PLAN.md`.
- Unrelated user changes: leave them alone and avoid staging or modifying those files.
- User changes in files you must edit: stop and ask before proceeding.
- Unknown generated or scratch files: inspect before deciding whether they belong in a checkpoint.

### 2. Checkpoint at every stable state

A stable state = compiles, relevant tests pass, and one coherent unit of the plan is done (typically one subtask from `task-decomposition-planner`). At each one:

```bash
git status --short
git add <intended-files-only>
git commit -m "S3: map TheHive payload fields (14/22) — enums pending, see .codex/STATE.md"
```

Commit message rules:

- Reference the subtask ID from `.codex/PLAN.md`.
- State what is DONE and what is explicitly NOT done.
- Meaningful messages only — a log of `wip`, `wip2`, `fix` is unresumable.

Then update `.codex/STATE.md` ("Currently in progress" / "Next actions") and tick `.codex/PLAN.md`. **The commit and the ledger update are one atomic ritual.**

### 3. Checkpoint before risk

Before any sweeping operation — codemod, mass rename, dependency upgrade, destructive migration script — commit first, even mid-subtask. Label it: `pre-risk: before sed rename of EnrichedAlert -> Alert`. This makes `git reset --hard` a one-command undo instead of an archaeology project.

### 4. Never checkpoint garbage

Do not commit known-broken states as checkpoints (exception: `pre-risk` snapshots, clearly labeled). If interrupted while broken, prefer a clear `.codex/STATE.md` note plus an explicit handoff. Use `git stash` only after inspecting `git status --short` and confirming you will not hide unrelated user changes; record any stash in `.codex/STATE.md`.

### 5. Cold-resume procedure

When resuming any prior session, reconstruct from artifacts, never from memory:

```bash
git log --oneline -10   # what was the last stable state?
git status              # anything uncommitted/stashed?
git stash list
```

Then read `.codex/STATE.md` and `.codex/PLAN.md`, reconcile all three, and state (briefly, to the user or in the ledger): *"Resuming at S4; S1-S3 verified per commits abc123..def456."* If artifacts disagree — uncommitted changes not described in the ledger — investigate and reconcile **before** writing new code. When in doubt about uncommitted work, stop and classify ownership instead of stashing blindly.

### 6. Landing the work

Before handoff: run the full verification pass (`self-verification-loop`), then review the branch history. Offer the user either the branch as-is (honest history) or a squashed/tidied version — their call, their repo conventions. Never force-push over anything you didn't create this session.

## Anti-patterns

- One giant commit at the end. That's not checkpointing; that's gambling the whole session on nothing going wrong.
- Commits without ledger updates. Six months of `git log` cannot tell you what "Next actions" were.
- Committing secrets, scratch files, or user changes because staging was reflexive. Inspect `git status --short`, then stage only intended files.
- Building on top of mystery uncommitted changes found at resume time.
