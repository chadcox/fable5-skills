---
name: codebase-cartographer
description: Build an explicit architecture map of a codebase before making multi-file changes or debugging non-local failures. Use this skill BEFORE any refactor touching 3+ files, any change to a shared interface, any debugging where the symptom's cause may live in a different module, or the first substantial task in an unfamiliar repo. Trigger especially when tempted to start editing immediately in a codebase you haven't mapped this session.
---

# Codebase Cartographer

## Why this exists

The hardest coding failures are non-local: the bug's cause is far from its symptom, or a refactor breaks a caller three modules away that nobody grepped for. Strong performance on cross-module work comes from actually holding the architecture. You can approximate that by building an **explicit, written map** before editing — and consulting it while you work.

Time spent mapping is not overhead. Fifteen minutes of cartography routinely saves hours of whack-a-mole.

## Workflow

### 1. Recon the territory

Before editing anything:

```bash
# Shape of the repo
tree -L 2 -I 'node_modules|.git|__pycache__|dist'  # or equivalent
# Entry points & wiring
rg -n "if __name__|def main|createApp|func main" -g <lang glob>
# Where do tests live, and how are they run?
ls tests/ ; rg -n "test|pytest|vitest|jest|mocha" Makefile package.json pyproject.toml 2>/dev/null
# Config & conventions
cat README* CONTRIBUTING* .editorconfig 2>/dev/null
```

Read the code that matters: the modules you'll touch, plus **one level up (their callers) and one level down (their dependencies)**. Skim, don't memorize — the map holds the details.

### 2. Write MAP.md

Persist the map (context compaction erases un-written maps):

```markdown
# MAP — <repo/task>

## Modules relevant to this task
- src/enrich/      — alert enrichment pipeline; entry: handler.py:process()
- src/notify/      — outbound Slack/email; consumes EnrichedAlert dataclass
- src/models.py    — shared dataclasses; CHANGES HERE FAN OUT WIDELY

## Call graph for the affected path
webhook -> handler.process() -> thehive_client.enrich() -> models.EnrichedAlert -> notify.slack.post()

## Interfaces / contracts I must not break silently
- EnrichedAlert fields consumed by notify/ AND by report_gen/ (rg confirmed: 2 consumers)
- REST /api/v1/alerts response schema — external consumers exist (SOAR playbooks)

## Conventions observed
- Errors: raise domain exceptions, caught at handler level; do NOT return None on failure
- Logging: structlog, keys are snake_case
- Tests: pytest, fixtures in conftest.py, integration tests need docker-compose up

## Blast radius of planned change
- Changing EnrichedAlert.severity type: 2 direct consumers + 6 test files (rg list below)
```

### 3. Verify the map with rg, not intuition

For every interface you plan to change, enumerate **all** consumers mechanically:

```bash
rg -n "EnrichedAlert" -g "*.py" -g "!*test*"   # then tests separately
```

The map records the rg results, not your guess. Hand this list to `multi-file-atomic-edits` when executing.

### 4. Use the map while debugging

For non-local bugs, walk the call graph in the map and bisect: where along the path does the data last look correct? Instrument at module boundaries (the map tells you where they are). Update the map when you discover the wiring differs from what you assumed — a corrected map is a debugging artifact worth more than the fix itself.

### 5. Feed the other skills

- Blast radius section → `task-decomposition-planner` subtask list
- Discovered constraints → `working-memory-ledger` invariants
- Consumer lists → `multi-file-atomic-edits`

## Anti-patterns

- Editing an interface after reading only the module that defines it. Callers are where breakage lives.
- Mapping in your head. Un-written maps don't survive compaction and can't be re-verified.
- Mapping the entire repo. Map the affected paths plus one hop — a map that takes an hour to build won't get built.
- Trusting file names over rg. `utils.py` lies.
