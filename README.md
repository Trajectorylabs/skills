# Trajectory skills

Agent skills for training and evaluating agents on [Trajectory](https://docs.trajectory.ai).
They teach a coding agent (Claude Code, Codex, Cursor and others) to do the common jobs
end to end with the Trajectory SDK and `tj` CLI: bring a benchmark to ready, run a
baseline eval, size and watch a training run, work out why reward is not moving, compare
runs, and write up what happened with evidence. Skills that spend money or change
anything state a plan and wait for your yes first; the investigation skills are read only.

## Before you install

The skills drive the Trajectory SDK, so add `trajectory-sdk` to your project and sign in
with `tj login`, or set `TRAJECTORY_API_KEY`. Everything else comes with the install
below, including the `trajectory` skill the others build on.

## Install

### Any agent, with `npx skills`

```bash
npx skills add Trajectorylabs/skills
```

Choose the skills and your agent when prompted, or take them all:

```bash
npx skills add Trajectorylabs/skills --skill '*' --agent claude-code --agent codex --agent cursor
```

To install only some, always include `trajectory`, which covers connecting, finding IDs
and linking resources for every other skill:

```bash
npx skills add Trajectorylabs/skills --skill trajectory --skill explain-run --skill compare-runs
```

`diagnose-reward` links to reference files in `explain-run` and `compare-runs`, and
`trajectory` links to `agent-feedback`, so install those together.

### Claude Code

```bash
claude plugin marketplace add Trajectorylabs/skills
claude plugin install trajectory@trajectory
```

### Codex

```bash
codex plugin marketplace add Trajectorylabs/skills
codex plugin add trajectory@trajectory
```

### Cursor

```bash
cursor-agent plugin marketplace add https://github.com/Trajectorylabs/skills
```

Then install `trajectory` from `/plugin`.

## Skills

| Skill | What it does |
| --- | --- |
| [`trajectory`](skills/trajectory/SKILL.md) | The base skill: connect to Trajectory, find the SDK's bundled guides, link what you report, and the ground rules for integrating a benchmark, evaluating, training and serving. |
| [`set-up-benchmark`](skills/set-up-benchmark/SKILL.md) | Take a benchmark from a repository to ready on Trajectory by packaging its tasks, harness and grader, uploading it to an agent, building its runtime images and checking readiness. |
| [`fix-benchmark`](skills/fix-benchmark/SKILL.md) | Work out why a Trajectory benchmark cannot launch or does not grade and fix it in the benchmark's source. |
| [`run-eval`](skills/run-eval/SKILL.md) | Run an evaluation on a Trajectory benchmark and read its result, from choosing the benchmark and model through an agreed plan and cost to the score. |
| [`start-training`](skills/start-training/SKILL.md) | Size, launch and watch a first training run on a Trajectory benchmark, with an agreed budget and a rule for when to stop. |
| [`explain-run`](skills/explain-run/SKILL.md) | Explain what happened in one Trajectory training or eval run, and why. |
| [`compare-runs`](skills/compare-runs/SKILL.md) | Compare two or more Trajectory training runs or eval runs and say which did better and why. |
| [`diagnose-reward`](skills/diagnose-reward/SKILL.md) | Investigate why a Trajectory training run's reward is flat, falling, noisy or suspiciously high, and what to try next. |
| [`write-run-report`](skills/write-run-report/SKILL.md) | Write up an analysis of one Trajectory training or eval run, or a comparison of several, as a report someone can act on and check. |
| [`capture-production-traces`](skills/capture-production-traces/SKILL.md) | Connect an existing application's agent to Trajectory so its real conversations are recorded as production trajectories on an agent. |
| [`download-model`](skills/download-model/SKILL.md) | Download a trained Trajectory checkpoint's weights, either its LoRA adapter or the full model with the adapter merged in, to a folder the person chooses. |
| [`agent-feedback`](skills/agent-feedback/SKILL.md) | Send feedback you have authorized about Trajectory's API, SDK, documentation or workflow to the Trajectory team. |

## Updates

The skills are written in Trajectory's own repositories and copied here;
[SOURCE.md](SOURCE.md) names where each comes from and the commit this copy was taken at.
To pick up a new copy:

- `npx skills`: `npx skills update`.
- Claude Code: `claude plugin marketplace update trajectory`, then `claude plugin update trajectory@trajectory`.
- Codex: `codex plugin marketplace upgrade trajectory`.
- Cursor: refresh the marketplace from `/plugin`.

Ask a running agent to reload its skills afterwards.

## Roadmap

- A hosted, read-only Trajectory MCP server is in review. The plugin will add it once it
  is available; until then the skills use the SDK and CLI only.

## Problems

Open an issue in this repository with the skill's name, your agent and what happened.
Pull requests that change files under `skills/` cannot be merged here, because that
folder is replaced on each refresh; describe the change in an issue instead.

## Licence

[Apache-2.0](LICENSE).
