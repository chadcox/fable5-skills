---
name: working-memory-ledger
description: Maintain a persistent <agent-artifacts>/STATE.md ledger of decisions, invariants, modified files, gotchas, and resumable checkpoints during any long or multi-file coding session. Use this skill on any task expected to span many tool calls, touch 3+ files, run long enough to risk context compaction, require git checkpoints, or be resumed later — large refactors, migrations, debugging marathons, multi-step infrastructure work. Trigger at task start and update continuously; also trigger when resuming ANY prior session.
---

# Working Memory Ledger

## Why this exists

On long tasks, context fills up and gets compacted; details from two hours ago are effectively gone. The result: re-discovering the same gotcha three times, contradicting an earlier design decision, forgetting which files were already migrated, or resuming at a guessed 80% and corrupting good work. Frontier models hold huge changes coherently; you can approximate that with an external ledger that is written eagerly, paired with git checkpoints when warranted, and re-read religiously.

The ledger is your durable working memory. **If it isn't in the ledger, assume future-you won't know it.**

Do **not** create a ledger for short tasks that can finish in one sitting, touch only a small known surface, and have no realistic context-compaction or resume risk. Use an inline checklist instead; start the ledger only when the task begins to sprawl.

## The ledger file

Create `<agent-artifacts>/STATE.md` next to `<agent-artifacts>/PLAN.md` at task start. If the repo already has a local convention for agent artifacts, use that instead. Structure:

```markdown
# STATE — <task name>
Last updated: <timestamp> — <what just happened>

## Decisions made (and WHY)
- D1: Using SQLAlchemy parameterized queries instead of raw SQL — SQLi remediation requirement.
- D2: Kept the legacy webhook behind FEATURE_SIR_FALLBACK flag — user wants 2-week overlap.

## Invariants / discovered constraints
- TheHive 5 SaaS rate-limits at 100 req/min — batch enrichments.
- LXC container is unprivileged; UID mapping means writes to /mnt/media must go through uid 101000 offset.
- Test suite requires REDIS_URL set or 14 tests silently skip.

## Files modified so far
- src/enrich/thehive_client.py — new API client (verified)
- src/enrich/handler.py — swapped SIR call (verified)
- src/notify/slack.py — UNTOUCHED but regression-tested

## Gotchas hit (so we don't hit them twice)
- G1: `pytest -n auto` deadlocks with the fake Redis fixture — run serial for tests/enrich/.
- G2: PSU on Linux drops env vars in New-PSUSchedule blocks — pass secrets via $Secret: scope.

## Currently in progress
- S3: payload field mapping — mapped 14/22 fields, remaining listed in `<agent-artifacts>/PLAN.md`

## Checkpoints
- abc1234 — S2 complete; tests/enrich passed; S3 not started
- pre-risk def5678 — before codemod rename of EnrichedAlert -> Alert

## Next actions
1. Finish field mapping (severity + TLP enums are non-obvious, see G-notes)
2. Regression-run Slack notification path
```

## Rules

### Write eagerly

Update `<agent-artifacts>/STATE.md` **immediately** after any of these, not "later":

- A design decision is made (with the *why* — future-you needs the rationale to avoid re-litigating it)
- A non-obvious constraint or environment quirk is discovered
- A file is modified and verified
- A gotcha costs you more than one attempt
- A subtask starts or finishes
- A git checkpoint is created, especially before a risky or sweeping operation

The "Last updated" line doubles as a heartbeat — one sentence of what just happened.

### Read religiously

Re-read `<agent-artifacts>/STATE.md` in full:

- Whenever resuming after any interruption or context compaction
- Before starting each new subtask
- Whenever you're surprised — surprise usually means you forgot a recorded invariant
- Before final handoff (the Decisions section becomes the skeleton of your summary to the user)

If you notice you're about to investigate something, first check whether `<agent-artifacts>/STATE.md` already has the answer.

### Checkpoint long work

Use git checkpoints only when the work is expected to run more than about 30 minutes, span multiple sittings, resume prior work, or perform a risky sweeping operation. Do not checkpoint merely because a task is gnarly.

A checkpoint is a git commit at a stable state plus updated `<agent-artifacts>/STATE.md` and `<agent-artifacts>/PLAN.md` progress marks. A commit without ledger context is a snapshot nobody can interpret.

Before checkpointing:

```bash
git status --short
git log --oneline -5
```

Classify any dirty tree first:

- Your own in-progress changes from this task: continue only if they match `<agent-artifacts>/STATE.md` / `<agent-artifacts>/PLAN.md`.
- Unrelated user changes: leave them alone and avoid staging or modifying those files.
- User changes in files you must edit: stop and ask before proceeding.
- Unknown generated or scratch files: inspect before deciding whether they belong in a checkpoint.

At each stable state:

```bash
git status --short
git add <intended-files-only>
git commit -m "S3: map TheHive payload fields (14/22); enums pending, see <agent-artifacts>/STATE.md"
```

Checkpoint before codemods, mass renames, dependency upgrades, destructive migration scripts, or other sweeping operations. Label pre-risk commits clearly. Do not commit known-broken states as checkpoints, except for explicit pre-risk snapshots.

### Keep it a ledger, not a log

Curate. Collapse resolved items, delete obsolete entries, keep it under ~150 lines. A bloated ledger doesn't get read, and an unread ledger is dead weight. History lives in git; `<agent-artifacts>/STATE.md` holds only what's needed to act correctly *right now*.

## Resuming a session

On any resume: read `<agent-artifacts>/STATE.md`, read `<agent-artifacts>/PLAN.md`, run `git log --oneline -10`, `git status`, and `git stash list`. Reconcile the artifacts before writing new code. If they disagree (for example, a file is modified but not in the ledger), investigate and fix the ledger first. Never resume from memory of the previous session.

## Anti-patterns

- Recording *what* was decided without *why*. The why is the part that prevents contradiction later.
- Updating the ledger in one big batch at the end. That's a eulogy, not working memory.
- Duplicating the plan. `<agent-artifacts>/PLAN.md` owns *what to do*; `<agent-artifacts>/STATE.md` owns *what we know and where we are*.
- Trusting your recollection over the ledger. The ledger was written by someone with more context than you currently have.
- One giant commit at the end of a multi-hour task. That is not checkpointing; it is gambling the whole session on nothing going wrong.
- Commits without ledger updates. Git history cannot tell future-you what the next action was.
