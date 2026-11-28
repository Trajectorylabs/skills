# Diffing run configs

A run's config is its `options` plus `bench_id`, `base_model_slug` and
`parent_checkpoint_id`, and for an eval `task_selection`, `total_rollouts`,
`training_run_id` and `step`. The [training guide](https://docs.trajectory.ai/guides/training.md) ("Training options") defines every
option; report values and cite it for meaning.

## How to diff

1. Flatten each config to one key per value, so a nested difference is not missed.
2. Line the runs up side by side, one row per key, in the same run order every time.
3. Treat "not set" as a value. An option one run set and another left to its default is a
   difference, and a default computed from other options (`min_trajectories_per_group`
   from `group_size`) can differ between runs that both left it unset.
4. Report only the rows that differ, with both values, and how many keys matched. Skip
   IDs, names and timestamps.
5. If a run's config could not be read, say so; do not diff against an empty one.

## Which differences matter

The first group makes the runs incomparable; the rest are candidate explanations.

| Group | Fields | Why it matters |
| --- | --- | --- |
| What was run | `bench_id`, `base_model_slug`, `parent_checkpoint_id`, `load_weights_only`; for evals `task_selection`, `total_rollouts` | Different tasks, grader, starting weights or sample sizes |
| How much was learned from | `num_steps`, `batch_size`, `group_size`, `min_trajectories_per_group`, `algorithm` | How many trajectories each update sees; compare at equal steps only when these agree |
| How big each update was | `learning_rate`, `lr_warmup_steps`, `lr_hold_steps`, `lr_min_ratio` | The usual reason one run climbs and another stalls or collapses |
| What was allowed to train | `train_on_*`, `max_staleness`, `density_aware_training` | Which trajectories reach the optimizer; shows in `used_groups` before it shows in reward |
| How rollouts were limited | `max_turns_per_trajectory`, `max_output_tokens_per_turn`, `max_output_tokens_per_trajectory`, `trajectory_timeout_seconds` | How rollouts end; check `termination_reasons` before crediting the model |
| How the model sampled | `temperature`, `top_p`, `top_k`, `reasoning_effort` | Behaviour and reward spread inside a group, with no training involved |
| How held-out reward was measured | every `val_*` option | Held-out rewards compare only when these agree |
| Throughput | `max_concurrent_trajectories`, `val_max_concurrent_trajectories`, `batch_timeout_seconds` | Speed and spend; rarely explains reward |

Several differences at once cannot be told apart from reward alone: say which one the step
summaries or trajectories support. No differences is a finding too: the runs diverged on
environment or run-to-run noise.
