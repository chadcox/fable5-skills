---
name: parallel-work-splitter
description: Decompose large coding jobs into independent workstreams with explicit interface contracts so they can run as parallel subagents or parallel Claude Code sessions, then merge cleanly. Use this skill when a task is large enough that serial execution would take hours AND contains genuinely independent parts — big migrations, multi-component features, sweeping repo-wide changes, or when the user asks to "split this up" or run work in parallel.
---

# Parallel Work Splitter

## Why this exists

The practical way to get whole-job, walk-away throughput from a smaller model is horizontal: split the job into independent streams, run several sessions/subagents at once, and merge. The failure mode isn't the parallelism — it's the split. Streams that secretly share files or contracts produce merge hell that costs more than the parallelism saved. This skill is about **cutting along real seams and freezing the contracts before anyone starts.**

Rule zero: **parallelize only what is actually independent.** When in doubt, keep it serial — a clean serial run beats a dirty parallel one.

## Workflow

### 1. Find the seams

From `.codex/PLAN.md` and `.codex/MAP.md`, group subtasks by the files/modules they touch. Candidate streams must satisfy:

- **Disjoint write-sets.** No two streams modify the same file. (Shared *reads* are fine.)
- **No hidden coupling** through generated code, schemas, lockfiles, or global config — the classic collision points. Lockfile-touching work (dependency changes) goes in exactly one stream, or serial.
- **Independently verifiable.** Each stream can run its own tests to green without another stream's output.

If two subtasks share a contract that isn't finished yet, they are one stream, or serialized.

### 2. Freeze the interface contracts FIRST

Anything two streams both depend on gets defined **before** the split, in a contracts step done serially: shared dataclasses/types, API schemas, function signatures, event formats, config keys. Write them down in `.codex/CONTRACTS.md` by default (or commit stub interfaces directly). During parallel execution, **contracts are frozen** — a stream that needs a contract change stops and escalates rather than unilaterally editing shared surface.

### 3. Write the work orders

Each stream gets a self-contained brief — assume the executing session has *zero* context beyond the repo and the brief:

```markdown
# Stream B — Notification consumers
- Branch: task/notify-consumers   Base: task/contracts (commit abc123)
- Scope (write-set): src/notify/**, tests/notify/** ONLY. Do not touch src/models.py.
- Contract: consume EnrichedAlert v2 as defined in .codex/CONTRACTS.md — treat as frozen.
- Requirements: R2, R4 from .codex/PLAN.md (restated here in full: ...)
- Definition of done: tests/notify green; zero-match `rg` search for legacy field names in scope.
- Escalate (stop, don't improvise) if: the contract is insufficient, or you need to write outside scope.
```

Each stream runs its own internal discipline (`self-verification-loop`, `working-memory-ledger` scoped to its branch).

### 4. Execute

- One git branch per stream, all cut from the same contracts commit.
- In Claude Code: parallel sessions (or git worktrees) per branch, one work order each; or subagents where available.
- The orchestrating session does not write code during the parallel phase — it monitors, answers escalations, and owns contract-change decisions (a contract change means pausing affected streams, updating `.codex/CONTRACTS.md`, rebasing).

### 5. Merge and integrate — the phase that actually bites

Merging is its own subtask with its own verification, never an afterthought:

1. Merge streams one at a time into an integration branch, running the **full** test suite after each merge (streams only ran their scoped tests; cross-stream breakage surfaces here).
2. Disjoint write-sets should make textual conflicts near-zero — a real conflict means the split was violated; find out why before resolving.
3. After all merges: full `self-verification-loop` + `scope-integrity-guard` reconciliation across the *original* requirements list — requirements can fall into the cracks *between* streams.

## Anti-patterns

- Splitting by requirement instead of by write-set. Two requirements that touch the same file are one stream.
- Letting streams "quickly fix" shared code. That's how disjoint write-sets die and merges explode.
- Vague work orders that assume the parallel session shares your context. It doesn't.
- Skipping the post-merge full-suite run because "each stream was green." Green in isolation proves nothing about composition.
- Parallelizing a 20-minute task. Orchestration overhead has a floor; below roughly an hour of serial work, just do it serially.
