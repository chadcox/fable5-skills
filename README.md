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

The repository also includes `AGENT_GUIDE.md`, a host-neutral sample global instruction file that shows how to route these skills across projects.

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

The included `AGENT_GUIDE.md` is a sample global routing file. Copy or merge its contents into the global instruction file for your agent host, such as `~/.claude/CLAUDE.md` for Claude Code or a global `AGENTS.md` for Codex-style agents.

For Claude Code, if you do not already have a global instruction file, install it directly:

```bash
mkdir -p ~/.claude
cp AGENT_GUIDE.md ~/.claude/CLAUDE.md
```

If you already have `~/.claude/CLAUDE.md`, avoid overwriting it. Copy this repo's sample beside it and import it from your existing global file:

```bash
mkdir -p ~/.claude
cp AGENT_GUIDE.md ~/.claude/fable5-skills-AGENT_GUIDE.md
grep -qxF '@~/.claude/fable5-skills-AGENT_GUIDE.md' ~/.claude/CLAUDE.md 2>/dev/null || \
  printf '\n@~/.claude/fable5-skills-AGENT_GUIDE.md\n' >> ~/.claude/CLAUDE.md
```

After installing in Claude Code, start Claude with `claude`. The skills should be available by name, such as `/effort-calibration`, and the global `CLAUDE.md` guidance should load at the start of each session.

For Codex-style agents, merge the same guidance into the relevant global `AGENTS.md` file and install the skill directories wherever that host expects reusable skills. For other coding-agent hosts, copy the same routing sections into that host's global instruction file.

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

## Using `AGENT_GUIDE.md`

`AGENT_GUIDE.md` is a sample global instruction file. It tells an agent to:

- Prefer available skills before inventing a workflow.
- Keep trivial tasks lightweight.
- Use persisted planning artifacts only when they materially prevent drift.
- Make surgical changes.
- Verify work before declaring success.

You can merge the relevant sections into Claude Code's global `CLAUDE.md`, a Codex-style global `AGENTS.md`, or another host-specific global instruction file.

## Artifact Paths

The workflows use durable agent artifacts when written state materially reduces drift. The model-agnostic names are:

- `<agent-artifacts>/PLAN.md` for task plans.
- `<agent-artifacts>/MAP.md` for architecture maps.
- `<agent-artifacts>/STATE.md` for working memory.

The included skills use `<agent-artifacts>/` as a placeholder, not a literal required directory. Hosts can map it to `.claude/`, `.codex/`, `.agents/`, or any project-local convention that keeps durable agent notes separate from source files.

## Repository Layout

```text
.
├── AGENT_GUIDE.md
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

Some of the general coding-agent guidance in `AGENT_GUIDE.md` was adapted from [`multica-ai/andrej-karpathy-skills`](https://github.com/multica-ai/andrej-karpathy-skills/blob/main/CLAUDE.md).

## Testing

Four small pilot evals have been run against Claude Code and Codex, comparing a baseline agent (no skills) against a treatment agent (this repo's skills plus the `AGENT_GUIDE.md` routing installed as `CLAUDE.md`). Full methodology, transcripts, and scoring are in `agent-artifacts/EVAL_RESULTS*.md`.

- On easy tasks (a typo fix, a one-line bug with a passing test, an environment-driven test failure, a two-file rename), baseline and treatment were indistinguishable across three model tiers (GPT-5.5 high, Sonnet 5, Haiku 4.5): 4/4 success in every condition, with treatment costing 25-60% more tokens depending on model tier and no correctness gain. See `EVAL_RESULTS.md`, `EVAL_RESULTS_CLAUDE.md`, `EVAL_RESULTS_HAIKU.md`.
- On harder adversarial fixtures built with a specific trap per skill (a spec requirement the visible test doesn't check, a misleading error message, a hidden untested consumer, a root-cause-vs-symptom bug, a dirty worktree, an ambiguous prompt), Sonnet 5's baseline still avoided every trap unaided (0/6 differential). Haiku 4.5's baseline fell into 2 of 6: it shipped a rename that silently broke an untested consumer script, and it invented an unrequested config value instead of asking. Treatment caught both. See `EVAL_RESULTS_ADVERSARIAL.md`.

Bottom line: these are pilot-scale results (single run per condition per task), not a validated benchmark. The one consistent finding so far is that the skills only changed an outcome when a weaker model (Haiku-tier) faced a task with a real trap in it — on easy tasks or with a stronger baseline model, the skills added token/cost overhead without measurable benefit. Don't treat either direction as proven; see the eval files for what would be needed to say more (repetition, more adversarial tasks, additional model tiers).

## Notes

Licensed under the MIT License. See [LICENSE](LICENSE).

These skills are operating procedures, not code libraries. They work best when the host agent is explicitly instructed to treat skills as mandatory workflows where applicable, not optional advice.

The included guidance intentionally favors correctness, verification, and scope control over raw speed. For very small tasks, `effort-calibration` keeps the process from becoming heavier than the work.

When durable agent artifacts are needed, use the project or host convention for `<agent-artifacts>`. The workflow depends on stable files, not on a specific directory name.
