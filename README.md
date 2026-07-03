# Fable5 Skills

Behavioral skills authored with the Fable5 model to help coding agents behave with more Fable-like discipline: calibrating effort, planning only when it pays off, making surgical changes, recovering from failures deliberately, and verifying before claiming success.

The skills are model-agnostic Markdown workflows. They are intended for agents that can discover and follow `SKILL.md` files, including Codex-style and Claude-style coding setups.

## What Is Included

Each directory is a standalone skill:

| Skill | Purpose |
| --- | --- |
| `effort-calibration` | Routes coding tasks into trivial, standard, or gnarly effort levels so the agent uses the right amount of process. |
| `self-verification-loop` | Requires concrete verification before reporting that work is complete. |
| `failure-recovery-protocol` | Turns failed commands, tests, and builds into hypothesis-driven debugging instead of blind retries. |
| `task-decomposition-planner` | Creates a short persisted plan for gnarly or drift-prone tasks. |
| `codebase-cartographer` | Builds a written map of relevant architecture before broad refactors or non-local debugging. |
| `working-memory-ledger` | Maintains `.codex/STATE.md` during long sessions so decisions, gotchas, and progress survive context loss. |
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

For Codex-style local skills, copy the skill directories into your skills folder:

```bash
mkdir -p ~/.codex/skills
cp -R effort-calibration self-verification-loop failure-recovery-protocol \
  task-decomposition-planner codebase-cartographer working-memory-ledger \
  scope-integrity-guard multi-file-atomic-edits long-horizon-checkpointing \
  parallel-work-splitter ~/.codex/skills/
```

## Install In Claude Code

Claude Code personal skills live in `~/.claude/skills/<skill-name>/SKILL.md`, where they are available across all projects. Install these skills there:

```bash
mkdir -p ~/.claude/skills
cp -R effort-calibration self-verification-loop failure-recovery-protocol \
  task-decomposition-planner codebase-cartographer working-memory-ledger \
  scope-integrity-guard multi-file-atomic-edits long-horizon-checkpointing \
  parallel-work-splitter ~/.claude/skills/
```

The included `CLAUDE.md` is intended to be global Claude Code guidance. If you do not already have a global Claude Code instruction file, install it directly:

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

After installing, start Claude Code with `claude`. The skills should be available by name, such as `/effort-calibration`, and the global `CLAUDE.md` guidance should load at the start of each session.

Claude Code's docs describe personal skills at `~/.claude/skills/` and user instructions at `~/.claude/CLAUDE.md`.

## Recommended Routing

Use `effort-calibration` as the first stop for coding work. It decides how much ceremony the task deserves:

- Trivial: execute directly, then run `self-verification-loop`.
- Standard: use a lightweight checklist, then run `self-verification-loop`.
- Gnarly: use the heavier stack: `task-decomposition-planner`, `codebase-cartographer`, `working-memory-ledger`, `scope-integrity-guard`, and final verification.

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

When durable agent artifacts are needed, the skills default to `.codex/PLAN.md`, `.codex/MAP.md`, `.codex/STATE.md`, and `.codex/CONTRACTS.md` so working notes do not clutter the project root. If a project already has its own artifact convention, use that instead.
