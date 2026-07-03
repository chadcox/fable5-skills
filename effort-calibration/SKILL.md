---
name: effort-calibration
description: Classify every incoming task as trivial, standard, or gnarly BEFORE starting, and apply the matching level of process — a manual reasoning-effort dial. Use this skill at the START of every coding task to decide how much planning, mapping, and verification rigor the task deserves; re-trigger mid-task whenever a "trivial" task reveals hidden depth or a "gnarly" task turns out simple.
---

# Effort Calibration

## Why this exists

Frontier models expose a reasoning-effort dial; you can approximate one behaviorally. Uncalibrated effort fails in both directions: **over-engineering** (`<agent-artifacts>/PLAN.md`, `<agent-artifacts>/MAP.md`, and 40 minutes of ceremony for a one-line fix — process fatigue that trains everyone to skip process) and **under-thinking** (diving into a subtle concurrency bug with the same energy as a typo fix, then thrashing). Ten seconds of explicit classification up front sets the right posture for everything downstream — and tells the other skills in this suite when to fire.

Routing rule: when multiple skills appear applicable, this skill sets the process budget. Other skill triggers describe relevance, but they do not automatically escalate the task to maximum ceremony unless the trigger is safety-critical: cross-file contract work, repeated failures, resumed work, risky sweeping operations, or a workflow the user explicitly requested.

## The three tiers

Classify before the first edit. State the tier in one line ("Treating this as standard: 2 files, existing test coverage.") so the user can veto.

### Tier 1 — Trivial

**Signature:** single file, single obvious change, no ambiguity, blast radius ≈ zero. Typo fixes, a log line, bumping a timeout, an isolated one-liner.

**Posture:** just do it. No `<agent-artifacts>/PLAN.md`, no `<agent-artifacts>/MAP.md`. Still non-negotiable: run/compile the change and re-read the diff (a 30-second `self-verification-loop`). Trivial does not mean unverified.

### Tier 2 — Standard

**Signature:** 1–3 files, clear requirements, known territory, moderate blast radius. Typical bug fixes, small features, adding a test, extending an existing pattern.

**Posture:** lightweight plan (a 5-line checklist is fine — inline, not necessarily a file), targeted verification (relevant tests + diff review), `<agent-artifacts>/STATE.md` only if the task starts sprawling.

### Tier 3 — Gnarly

**Signature:** any of — multi-file/cross-module, ambiguous requirements, unfamiliar codebase, concurrency/security/data-migration territory, long horizon, high blast radius, or "we've tried to fix this before."

**Posture:** full discipline stack — `task-decomposition-planner`, `codebase-cartographer`, `working-memory-ledger`, and full `self-verification-loop` with an adversarial pass. Use the planner's boundary checks to preserve scope, and use the ledger's checkpoint guidance when the work is multi-hour, resumed, spans multiple sittings, or is about to perform a risky sweeping operation. Deliberately slow down: read more code than feels necessary before the first edit, and explicitly ask *"what would a senior engineer flag in this approach?"* before committing to a design.

## Classification heuristics

When torn between tiers, these push you up a tier:

- The word "just" in your own reasoning ("I'll just change...") — famous last words; check blast radius.
- Anything touching auth, crypto, money, deletion, migrations, or prod config.
- You can't name the test that will prove it works.
- The symptom is far from any code you plan to touch.
- The task arrived as "quick question" but contains the word "and" twice.

When torn, err **up**. The cost asymmetry is brutal: over-classifying wastes minutes; under-classifying wastes hours and ships bugs.

## Recalibrate on evidence

Classification is a prior, not a vow:

- **Escalate immediately** when a Tier 1/2 task surprises you: the "one-liner" fix touches a shared interface, the second consumer appears, the first fix attempt fails. Say so ("this is deeper than it looked — switching to full process"), then bring in the heavier skills. Sunk-cost momentum ("I'm almost done, no need to plan now") is the failure mode; two failed attempts = mandatory escalation review.
- **De-escalate** when a feared task collapses ("the migration is actually 3 call sites") — note it and finish light. Don't perform ceremony for its own sake.

## Anti-patterns

- Classifying everything Tier 2 to avoid thinking about it. The tails are where calibration pays.
- Letting the user's framing set the tier. "Quick fix?" is a hope, not a classification — classify on the work, not the wording.
- Silently escalating effort without telling the user, so a "5-minute" task goes dark for an hour. Recalibration is said out loud.
- Skipping verification on Tier 1. The tier controls planning depth, never whether you verify.
