---
name: explain-run
description: Explain what happened in one Trajectory training or eval run, and why. Use when someone asks how a run is going, why it failed or stalled, why its reward moved or stayed flat, or what its trajectories show.
metadata:
  artemis-requires: none
  artemis-user-invocable: "true"
  artemis-argument-hint: "[run ID or name]"
  artemis-summary: "Explain what happened in a run"
---

# Explain a run

Every tool result carries the platform URL of each resource it names: link with those.
Read only. Never cancel, resume or relaunch a run while explaining it.

## 1. Pin down the run

- A training run ID starts with `trn_`, an eval run ID with `evr_`, a trajectory ID with
  `traj_`. Without an ID, find the benchmark's `bench_id` with `list_benchmarks`
  (search by name), then the run with `list_training_runs` or
  `list_eval_runs` on it (`status=["running"]` for active runs), and confirm the
  match before going on.
- Read the run: `get_training_run` or `get_eval_run`. Note
  status, `failure`, base model, benchmark, `created_by`, and the options it set.
- Read progress: `get_training_progress` (completed and budgeted steps) or
  `get_eval_progress` (waiting, running, scored, failed and cancelled rollouts).

If the run failed, lead with `failure.code` and `failure.message`, quoted. Do not guess a
cause the record does not show.

## 2. Training run: follow reward through the steps

1. `get_reward_history`: mean reward and trajectory count per step.
   Find where reward rose, fell or went flat, and how many trajectories each point rests on.
   `pending_count` above zero means the step is still being graded.
2. `get_held_out_rewards`: validation reward per step. Compare it
   only for rounds with `round_completed` true and `completed_samples` equal to
   `expected_samples`. Training reward rising while held-out stays flat is worth saying.
3. `get_step_summaries` for the steps around the change: outcome
   counts, `termination_reasons`, turns, tokens, tool failures, and group signal. Read
   [references/step-summaries.md](references/step-summaries.md) to interpret them.
4. `list_trajectories` with `training_run_id` for examples at the step that matters
   (`policy_step`), then `get_trajectory` for its reward components and ending, and
   `get_trajectory_steps` for what the agent did. Pick one or two that show the
   pattern, not the first ones listed.

## 3. Eval run: account for every rollout

1. From `get_eval_progress`, report scored, failed, cancelled and unfinished
   rollouts against `total_rollouts` before any mean. A mean over half the rollouts is a
   partial result; say so.
2. `final_reward_mean` is authoritative only when the eval completed;
   `provisional_reward_mean` covers scored rollouts so far.
3. If it evaluated a checkpoint, `training_run_id` and `step` on the eval run name it.
4. `list_trajectories` with `eval_run_id` for failed or low-reward examples, then
   `get_trajectory_steps` to see why.

## 4. Answer

- First line: the answer (state, outcome, and the main reason), with the run's link.
- Then the evidence: the numbers that support it, each with its step, and links to the one
  or two trajectories that show it.
- Say what you could not see: missing held-out rounds, steps still pending, data an
  endpoint does not return.
- Separate observation from inference. "Reward fell at step 40" is observed; "the new
  grader is stricter" is a guess unless a trajectory shows it.
