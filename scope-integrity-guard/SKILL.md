---
name: scope-integrity-guard
description: Prevent scope creep and silently dropped requirements during long tasks by re-anchoring against the original request at every phase boundary. Use this skill during ANY multi-subtask coding session — trigger at each subtask boundary, whenever tempted to make an unrequested "improvement", whenever the task has evolved through follow-up messages, and always immediately before final handoff.
---

# Scope Integrity Guard

## Why this exists

Long sessions bleed scope in **both** directions:

- **Creep**: unrequested refactors, "while I'm here" cleanups, gold-plating — each one adds risk, review burden, and time the user didn't ask for.
- **Shrink**: requirements silently dropped. The user asked for A, B, and C; three hours later they receive A, most of B, and no mention of C. This is the classic long-horizon failure, and it's worse than creep because the user discovers it in production.

The defense is mechanical, not motivational: **re-read the original request, out loud, at every phase boundary.**

## Workflow

Do **not** use this skill for a single-step task with no meaningful phase boundary, no follow-up requirements, and no drift risk. A final `self-verification-loop` requirement check is enough for those cases.

### 1. Pin the scope at task start

In `.codex/PLAN.md` (see `task-decomposition-planner`), the Requirements section is the scope contract. Include implicit-but-obvious requirements, and mark anything ambiguous. If the user adds requirements mid-session via follow-up messages, **append them to the Requirements list immediately** — scope that lives only in chat scrollback is scope that gets dropped.

### 2. The boundary ritual

At every subtask boundary (and after any long debugging detour), run this 60-second check:

1. Re-read the user's original message(s) — the actual text, not your memory of it.
2. Walk the Requirements list. Mark each: `done+verified` / `in progress` / `not started` / `at risk`.
3. Ask: is anything I just built **not** traceable to a requirement? If so, it's creep — see below.
4. Ask: did the detour I just took orphan any requirement? Re-slot it into Next actions.

### 3. Handling creep impulses

When you notice a tempting improvement outside scope (messy code nearby, a deprecated pattern, a missing test for unrelated code):

- **Default: don't do it.** Record it in `.codex/STATE.md` under a `## Noticed (out of scope)` list.
- **Exception — do it if:** it's required for the in-scope change to work correctly, or it's a genuine safety issue (e.g., you found a credential in the repo, an injection vulnerability in the code path you're editing). Do the minimum, flag it prominently.
- **At handoff**, present the Noticed list to the user as recommendations. This converts creep into value: the user gets the observations *and* control over what happens next.

### 4. Handling shrink pressure

When time/context runs low, the temptation is to quietly narrow the task. Never narrow silently. Legitimate options, in order of preference:

1. Checkpoint (see `long-horizon-checkpointing`) and tell the user what remains.
2. Propose an explicit descope: "C is larger than expected because X — ship A+B now and do C separately?"
3. If the user is unavailable and a call must be made, complete the highest-value requirements fully rather than all requirements partially — and say exactly which is which at handoff.

### 5. The handoff audit

Immediately before final delivery, produce the scope reconciliation and include it in your report:

```
## Scope reconciliation
- R1 (TheHive integration): DONE — verified (tests + manual run)
- R2 (Slack notifications preserved): DONE — regression-tested
- R3 (remove legacy SIR path): DESCOPED per your message 14:32 — feature-flagged instead
- Out-of-scope items noticed (not touched): raw SQL in report_gen.py; TODO auth check in admin route
```

Every requirement accounted for, every deviation attributed to an explicit decision. Nothing silent.

## Anti-patterns

- "While I was in there, I also rewrote..." — unrequested rewrites in a delivered diff.
- Delivering 4 of 5 requirements with a summary that only mentions the 4.
- Letting follow-up-message requirements live only in scrollback.
- Treating scope questions as interruptions. A 1-line descope question is cheaper than a wrong guess in either direction.
