# Fable5 Skills

Behavioral skills authored with the Fable5 model to help coding agents behave with more Fable-like discipline: calibrating effort, planning only when it pays off, making surgical changes, recovering from failures deliberately, and verifying before claiming success.

The skills are model-agnostic Markdown workflows. They are intended for any coding agent that can discover and follow `SKILL.md` files.

The default set is curated for usefulness, not for a fixed count. Earlier versions used ten skills as an arbitrary starting point; related long-task workflows have since been merged so the core install stays lean.

## What Is Included

Each top-level skill directory is a standalone skill:

| Skill | Purpose |
| --- | --- |
| `effort-calibration` | Routes coding tasks into trivial, standard, or gnarly effort levels so the agent uses the right amount of process. |
| `self-verification-loop` | Requires concrete verification before reporting that work is complete. |
| `failure-recovery-protocol` | Turns failed commands, tests, and builds into hypothesis-driven debugging instead of blind retries. |
| `task-decomposition-planner` | Creates a short persisted plan for gnarly or drift-prone tasks and preserves scope at phase boundaries. |
| `codebase-cartographer` | Builds a written map of relevant architecture before broad refactors or non-local debugging. |
| `working-memory-ledger` | Maintains a durable state ledger and checkpoint guidance during long sessions so decisions, gotchas, and progress survive context loss. |
| `multi-file-atomic-edits` | Handles cross-file contract changes, renames, schemas, and config edits as one coherent batch. |

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
  multi-file-atomic-edits ~/.codex/skills/
```

For Claude Code personal skills:

Claude Code personal skills live in `~/.claude/skills/<skill-name>/SKILL.md`, where they are available across all projects. Install these skills there:

```bash
mkdir -p ~/.claude/skills
cp -R effort-calibration self-verification-loop failure-recovery-protocol \
  task-decomposition-planner codebase-cartographer working-memory-ledger \
  multi-file-atomic-edits ~/.claude/skills/
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
- Gnarly: use the heavier stack: `task-decomposition-planner`, `codebase-cartographer`, `working-memory-ledger`, and final verification.

When multiple skills appear applicable, `effort-calibration` is the routing authority for process budget. Individual skill triggers describe when a workflow is relevant; they do not override the calibrated tier unless the task is a safety-critical trigger such as cross-file contract work, repeated failures, resumed work, or a user explicitly requests that workflow.

| Scenario | Route |
| --- | --- |
| Trivial coding task | Classify with `effort-calibration`, execute directly, then use `self-verification-loop`. Do not create durable artifacts. |
| Standard task | Use an inline checklist and targeted verification; create durable artifacts only if the task expands or drift risk becomes real. |
| Gnarly task | Use `task-decomposition-planner`, `codebase-cartographer`, `working-memory-ledger`, and final `self-verification-loop`. |
| Cross-file contract change | Use `multi-file-atomic-edits` before editing. |
| Failed command, test, build, or tool call | Use `failure-recovery-protocol` for non-trivial failures; always use it after the same command or fix attempt fails twice. |
| Resumed, multi-hour, or risky sweeping work | Use `working-memory-ledger` checkpoint guidance. |

Always use:

- `self-verification-loop` before reporting completion.
- `failure-recovery-protocol` after any non-trivial failed command, test, or build.
- `multi-file-atomic-edits` before cross-file renames, signature changes, schema changes, or config key changes.

Use only when warranted:

- `working-memory-ledger` checkpoint guidance for multi-hour, resumed, or risky sweeping work.

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

The included skills use `<agent-artifacts>/` as a placeholder, not a literal required directory. Hosts can map it to `.claude/`, `.codex/`, `.agents/`, or any project-local convention that keeps durable agent notes separate from source files.

## Repository Layout

```text
.
├── CLAUDE.md
├── README.md
├── codebase-cartographer/
├── effort-calibration/
├── failure-recovery-protocol/
├── multi-file-atomic-edits/
├── self-verification-loop/
├── task-decomposition-planner/
└── working-memory-ledger/
```

## Attribution

Some of the general coding-agent guidance in `CLAUDE.md` was adapted from [`multica-ai/andrej-karpathy-skills`](https://github.com/multica-ai/andrej-karpathy-skills/blob/main/CLAUDE.md).

## Notes

Licensed under the MIT License. See [LICENSE](LICENSE).

No formal testing has been completed to prove these skills are effective at changing model behavior or improving outcomes. They are provided as-is, with no guarantee of correctness, fitness for a particular model, or suitability for any workflow. Your mileage may vary.

These skills are operating procedures, not code libraries. They work best when the host agent is explicitly instructed to treat skills as mandatory workflows where applicable, not optional advice.

The included guidance intentionally favors correctness, verification, and scope control over raw speed. For very small tasks, `effort-calibration` keeps the process from becoming heavier than the work.

When durable agent artifacts are needed, use the project or host convention for `<agent-artifacts>`. The workflow depends on stable files, not on a specific directory name.
