---
name: multi-file-atomic-edits
description: Execute cross-cutting code changes (renames, signature changes, interface/schema changes, dependency swaps) as one coherent atomic batch instead of file-by-file drift. Use this skill whenever a change affects a symbol, contract, or pattern used in more than one file — trigger BEFORE the first edit of any rename, API signature change, dataclass/schema field change, config key change, or library replacement.
---

# Multi-File Atomic Edits

## Why this exists

The classic mess left behind by file-by-file editing: an interface changed in three places, forgotten in two, the codebase now half-migrated and broken in ways that surface one at a time. Cross-cutting changes must be treated as **one logical transaction**: enumerate every affected site first, change them all as a batch, verify as a unit.

Iron rule: **search first, edit second.** Prefer `rg` for codebase search. Never begin a cross-cutting edit until the complete list of affected sites exists as text you can check off.

## Workflow

Do **not** use this skill for a local single-file edit, an isolated behavior change, or a refactor with no shared symbol, schema, config key, serialized field, dependency, or public contract. Use normal scoped editing plus `self-verification-loop` instead.

### 1. Define the contract change precisely

Write one line: *"`EnrichedAlert.severity: int` becomes `severity: Severity` (enum); all producers and consumers must convert."* If you can't state the change in one line, it's multiple changes — split them and do them serially, each atomically.

### 2. Enumerate ALL affected sites mechanically

```bash
rg -n "severity" src/ tests/ -g "*.py"        # broad first
rg -n "EnrichedAlert\(" src/ tests/ -g "*.py" # constructors
rg -n "severity" config/ docs/ *.md           # configs & docs too
```

Also hunt non-obvious consumers: string-based access (`getattr`, `dict["severity"]`, serialized JSON, DB columns, API schemas, log parsers, dashboards, .yaml/.toml keys, PowerShell splats). Cross-check with `<agent-artifacts>/MAP.md` if `codebase-cartographer` ran. Record the complete site list (file:line) in `<agent-artifacts>/STATE.md` or a scratch checklist. **This list is the definition of done.**

### 3. Order the batch for continuous compilability where possible

Preferred order: (1) change the definition, (2) update all producers, (3) update all consumers, (4) update tests, (5) update docs/configs. In dynamic languages where nothing "fails to compile," the checklist is your only guardrail — lean on it harder.

For large/risky batches, consider the two-phase (expand/contract) pattern: add the new field/signature alongside the old, migrate consumers, then remove the old. Ask the user before choosing two-phase if it changes what gets delivered.

### 4. Execute as a single batch

- Checkpoint first using `working-memory-ledger` guidance when the batch is large or risky.
- Work straight through the checklist, ticking each site. **Do not interleave unrelated work** mid-batch — a half-done atomic edit plus a detour equals a lost thread.
- For mechanical renames, prefer tooling (`rg -l "old" | xargs sed -i`, IDE rename, codemod) over 40 hand-edits — but *always* review the resulting diff hunk-by-hunk; mechanical tools rename the substring in comments, strings, and unrelated symbols with enthusiasm.

### 5. Verify as a unit

Only after the entire checklist is ticked:

```bash
rg -n "old_symbol" . -g <glob>   # MUST return zero matches (or only intentional remnants)
<build / type-check / full test suite>
```

The zero-match search is non-negotiable — it catches the site the enumeration missed. Then run `self-verification-loop`. Commit the batch as **one commit**: `"rename EnrichedAlert.severity int->Severity enum across 2 producers, 6 consumers, 6 test files"`.

### 6. If interrupted mid-batch

A half-done batch is the worst state to abandon. Either finish the batch or revert to the pre-risk checkpoint — never leave it half-migrated. If you truly must stop, record the checklist with tick-state in `<agent-artifacts>/STATE.md`. Use `git stash` only after confirming it will not hide unrelated user changes.

## Anti-patterns

- Editing call sites as tests fail, one by one. Tests are the verification, not the discovery mechanism.
- Trusting the IDE/compiler to find all sites in dynamic code, templates, configs, and serialized formats.
- Bundling a rename with behavior changes in one batch/commit. Reviewers can't see the behavior change inside 40 files of rename noise — separate them.
- Declaring done without the zero-match search.
