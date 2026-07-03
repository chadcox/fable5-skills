# Fable5 Skills

Behavioral skills authored with the Fable5 model to help coding agents behave with more Fable-like discipline: calibrating effort, planning only when it pays off, making surgical changes, recovering from failures deliberately, and verifying before claiming success.

The skills are model-agnostic Markdown workflows. They are intended for any coding agent that can discover and follow `SKILL.md` files.

## What Is Included

Each directory is a standalone skill:

| Skill | Purpose |
| --- | --- |
| `effort-calibration` | Routes coding tasks into trivial, standard, or gnarly effort levels so the agent uses the right amount of process. |
| `self-verification-loop` | Requires concrete verification before reporting that work is complete. |
| `failure-recovery-protocol` | Turns failed commands, tests, and builds into hypothesis-driven debugging instead of blind retries. |
| `task-decomposition-planner` | Creates a short persisted plan for gnarly or drift-prone tasks. |
| `codebase-cartographer` | Builds a written map of relevant architecture before broad refactors or non-local debugging. |
| `working-memory-ledger` | Maintains a durable state ledger during long sessions so decisions, gotchas, and progress survive context loss. |
| `scope-integrity-guard` | Re-checks the original request at phase boundaries to avoid scope creep or dropped requirements. |
| `multi-file-atomic-edits` | Handles cross-file contract changes, renames, schemas, and config edits as one coherent batch. |
| `long-horizon-checkpointing` | Uses git checkpoints and state files to make long coding sessions resumable. |
| `parallel-work-splitter` | Splits large jobs into independent workstreams with explicit write sets and contracts. |

The repository also includes `CLAUDE.md`, a sample global instruction file that shows how to route these skills for Claude across projects.

## Install

Clone the repository:

```bash
git clone https://github.com/chadcox/fable5-skills.git
cd fable5-skills
```

Install the skill directories into the skill folder used by your coding-agent host. The target path is host-specific; these examples cover common personal skill locations.

For Codex-style local skills:

```bash
mkdir -p ~/.codex/skills
cp -R effort-calibration self-verification-loop failure-recovery-protocol \
  task-decomposition-planner codebase-cartographer working-memory-ledger \
  scope-integrity-guard multi-file-atomic-edits long-horizon-checkpointing \
  parallel-work-splitter ~/.codex/skills/
```

For Claude Code personal skills:

Claude Code personal skills live in `~/.claude/skills/<skill-name>/SKILL.md`, where they are available across all projects. Install these skills there:

```bash
mkdir -p ~/.claude/skills
cp -R effort-calibration self-verification-loop failure-recovery-protocol \
  task-decomposition-planner codebase-cartographer working-memory-ledger \
  scope-integrity-guard multi-file-atomic-edits long-horizon-checkpointing \
  parallel-work-splitter ~/.claude/skills/
```

## Install The Routing Guidance

The included `CLAUDE.md` is a sample global routing file. It is written for Claude Code, but the routing rules are portable to other global instruction files such as `AGENTS.md`.

If you do not already have a global Claude Code instruction file, install it directly:

```bash
mkdir -p ~/.claude
cp CLAUDE.md ~/.claude/CLAUDE.md
```

If you already have `~/.claude/CLAUDE.md`, avoid overwriting it. Copy this repo's sample beside it and import it from your existing global file:

```bash
mkdir -p ~/.claude
cp CLAUDE.md ~/.claude/fable5-skills-CLAUDE.md
grep -qxF '@~/.claude/fable5-skills-CLAUDE.md' ~/.claude/CLAUDE.md 2>/dev/null || \
  printf '\n@~/.claude/fable5-skills-CLAUDE.md\n' >> ~/.claude/CLAUDE.md
```

After installing in Claude Code, start Claude with `claude`. The skills should be available by name, such as `/effort-calibration`, and the global `CLAUDE.md` guidance should load at the start of each session.

For other coding-agent hosts, copy the same routing sections into that host's global instruction file and install the skill directories wherever that host expects reusable skills.

## Recommended Routing

Use `effort-calibration` as the first stop for coding work. It decides how much ceremony the task deserves:

- Trivial: execute directly, then run `self-verification-loop`.
- Standard: use a lightweight checklist, then run `self-verification-loop`.
- Gnarly: use the heavier stack: `task-decomposition-planner`, `codebase-cartographer`, `working-memory-ledger`, `scope-integrity-guard`, and final verification.

When multiple skills appear applicable, `effort-calibration` is the routing authority for process budget. Individual skill triggers describe when a workflow is relevant; they do not override the calibrated tier unless the task is a safety-critical trigger such as cross-file contract work, repeated failures, resumed work, or a user explicitly requests that workflow.

| Scenario | Route |
| --- | --- |
| Trivial coding task | Classify with `effort-calibration`, execute directly, then use `self-verification-loop`. Do not create durable artifacts. |
| Standard task | Use an inline checklist and targeted verification; create durable artifacts only if the task expands or drift risk becomes real. |
| Gnarly task | Use `task-decomposition-planner`, `codebase-cartographer`, `working-memory-ledger`, `scope-integrity-guard`, and final `self-verification-loop`. |
| Cross-file contract change | Use `multi-file-atomic-edits` before editing. |
| Failed command, test, build, or tool call | Use `failure-recovery-protocol` for non-trivial failures; always use it after the same command or fix attempt fails twice. |
| Resumed, multi-hour, or risky sweeping work | Use `long-horizon-checkpointing`. |
| Large independent workstreams | Use `parallel-work-splitter` only when write sets are genuinely independent. |

Always use:

- `self-verification-loop` before reporting completion.
- `failure-recovery-protocol` after any non-trivial failed command, test, or build.
- `multi-file-atomic-edits` before cross-file renames, signature changes, schema changes, or config key changes.

Use only when warranted:

- `long-horizon-checkpointing` for multi-hour or resumed work.
- `parallel-work-splitter` for genuinely independent large workstreams.

## Using `CLAUDE.md`

`CLAUDE.md` is a sample global Claude instruction file. It tells an agent to:

- Prefer available skills before inventing a workflow.
- Keep trivial tasks lightweight.
- Use persisted planning artifacts only when they materially prevent drift.
- Make surgical changes.
- Verify work before declaring success.

You can merge the relevant sections into your global `CLAUDE.md`, or adapt them for another global agent instruction file such as `AGENTS.md`.

## Artifact Paths

The workflows use durable agent artifacts when written state materially reduces drift. The model-agnostic names are:

- `<agent-artifacts>/PLAN.md` for task plans.
- `<agent-artifacts>/MAP.md` for architecture maps.
- `<agent-artifacts>/STATE.md` for working memory.
- `<agent-artifacts>/CONTRACTS.md` for frozen parallel-work contracts.

The included skills use `.codex/` as their default artifact directory because it keeps working notes out of the project root in Codex-style setups. That path is a convention, not a requirement. Claude, Codex, and other hosts can map `<agent-artifacts>` to `.claude/`, `.codex/`, `.agents/`, or a project-local convention.

## Repository Layout

```text
.
├── CLAUDE.md
├── README.md
├── codebase-cartographer/
├── effort-calibration/
├── failure-recovery-protocol/
├── long-horizon-checkpointing/
├── multi-file-atomic-edits/
├── parallel-work-splitter/
├── scope-integrity-guard/
├── self-verification-loop/
├── task-decomposition-planner/
└── working-memory-ledger/
```

## Notes

No formal testing has been completed to prove these skills are effective at changing model behavior or improving outcomes. They are provided as-is, with no guarantee of correctness, fitness for a particular model, or suitability for any workflow. Your mileage may vary.

These skills are operating procedures, not code libraries. They work best when the host agent is explicitly instructed to treat skills as mandatory workflows where applicable, not optional advice.

The included guidance intentionally favors correctness, verification, and scope control over raw speed. For very small tasks, `effort-calibration` keeps the process from becoming heavier than the work.

When durable agent artifacts are needed, use the project or host convention for `<agent-artifacts>`. The workflow depends on stable files, not on a specific directory name.
