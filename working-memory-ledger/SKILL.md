---
name: working-memory-ledger
description: Maintain a persistent .codex/STATE.md ledger of decisions, invariants, modified files, and gotchas during any long or multi-file coding session. Use this skill on any task expected to span many tool calls, touch 3+ files, run long enough to risk context compaction, or be resumed later — large refactors, migrations, debugging marathons, multi-step infrastructure work. Trigger at task start and update continuously; also trigger when resuming ANY prior session.
---

# Working Memory Ledger

## Why this exists

On long tasks, context fills up and gets compacted; details from two hours ago are effectively gone. The result: re-discovering the same gotcha three times, contradicting an earlier design decision, or forgetting which files were already migrated. Frontier models hold huge changes coherently; you can approximate that with an external ledger that is written eagerly and re-read religiously.

The ledger is your durable working memory. **If it isn't in the ledger, assume future-you won't know it.**

Do **not** create a ledger for short tasks that can finish in one sitting, touch only a small known surface, and have no realistic context-compaction or resume risk. Use an inline checklist instead; start the ledger only when the task begins to sprawl.

## The ledger file

Create `.codex/STATE.md` next to `.codex/PLAN.md` at task start. If the repo already has a local convention for agent artifacts, use that instead. Structure:

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
- S3: payload field mapping — mapped 14/22 fields, remaining listed in `.codex/PLAN.md`

## Next actions
1. Finish field mapping (severity + TLP enums are non-obvious, see G-notes)
2. Regression-run Slack notification path
```

## Rules

### Write eagerly

Update `.codex/STATE.md` **immediately** after any of these, not "later":

- A design decision is made (with the *why* — future-you needs the rationale to avoid re-litigating it)
- A non-obvious constraint or environment quirk is discovered
- A file is modified and verified
- A gotcha costs you more than one attempt
- A subtask starts or finishes

The "Last updated" line doubles as a heartbeat — one sentence of what just happened.

### Read religiously

Re-read `.codex/STATE.md` in full:

- Whenever resuming after any interruption or context compaction
- Before starting each new subtask
- Whenever you're surprised — surprise usually means you forgot a recorded invariant
- Before final handoff (the Decisions section becomes the skeleton of your summary to the user)

If you notice you're about to investigate something, first check whether `.codex/STATE.md` already has the answer.

### Keep it a ledger, not a log

Curate. Collapse resolved items, delete obsolete entries, keep it under ~150 lines. A bloated ledger doesn't get read, and an unread ledger is dead weight. History lives in git (see `long-horizon-checkpointing`); `.codex/STATE.md` holds only what's needed to act correctly *right now*.

## Resuming a session

On any resume: read `.codex/STATE.md`, read `.codex/PLAN.md`, run `git log --oneline -10` and `git status`. Reconcile the three. If they disagree (e.g., a file is modified but not in the ledger), investigate and fix the ledger before writing any new code. Never resume from memory of the previous session — resume from the artifacts.

## Anti-patterns

- Recording *what* was decided without *why*. The why is the part that prevents contradiction later.
- Updating the ledger in one big batch at the end. That's a eulogy, not working memory.
- Duplicating the plan. `.codex/PLAN.md` owns *what to do*; `.codex/STATE.md` owns *what we know and where we are*.
- Trusting your recollection over the ledger. The ledger was written by someone with more context than you currently have.
