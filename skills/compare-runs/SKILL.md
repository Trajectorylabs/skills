---
name: compare-runs
description: Compare two or more Trajectory training runs or eval runs and say which did better and why. Use when someone asks whether a change helped, which run or checkpoint is better, or why two runs on the same benchmark diverged.
---

# Compare runs

Builds on the `trajectory` skill: connect, find IDs and link resources the way it says.
Read only. To explain a single run in depth, use the `explain-run` skill.

## 1. Check the runs are comparable

Read each run with `client.training.runs.retrieve` or `client.evals.runs.retrieve`, then
list what differs before comparing any number:

- Benchmark (`bench_id`). Different benchmarks, or versions of one, are not comparable;
  say so and stop unless the person only wants a side-by-side.
- Base model, parent checkpoint (`parent_checkpoint_id`), and for eval runs the evaluated
  training run and `step`.
- Options: diff the recorded `options` field by field. Name each difference; it is the
  first candidate explanation. [references/config-diff.md](references/config-diff.md) says
  how to diff and which fields matter.
- For eval runs, `task_selection` and `total_rollouts`: different task sets or sample
  counts make means incomparable.
- Status: compare a running or failed run only up to its last completed step, and say so.

## 2. Compare training runs

1. `client.training.rewards.list_history` for each run. Line them up by step, not by time.
   Report reward at the same steps (start, a few points in between, the last step both
   reached), and the step where they part ways.
2. `client.training.rewards.list_held_out_rewards` for each: compare only rounds with
   `round_completed` true and matching sample counts. Held-out reward decides which run
   generalizes better; training reward alone does not.
3. `client.training.runs.list_step_summaries` around the divergence: differences in
   `no_signal_groups`, `used_groups`, `termination_reasons`, turns or tokens usually explain
   it better than the reward curve does.
4. Show one trajectory from each run on the same task (`task_id`) when it makes the
   difference concrete: `client.trajectories.list`, then `client.trajectories.steps.list`.

## 3. Compare eval runs

1. `client.evals.runs.progress` for each: scored, failed and cancelled counts first. A run
   with many failed rollouts has a mean over fewer tasks.
2. Then the means: `final_reward_mean` for completed evals, otherwise
   `provisional_reward_mean`, labelled as provisional.
3. With a few dozen rollouts, a difference of a few points can be noise. Give the counts so
   the reader can judge, and do not call a small gap a win.
4. For a per-task view, list each run's trajectories with `client.trajectories.list` and
   match them on `task_id`: which tasks flipped from failing to passing, and back.

## 4. Answer

- First line: which run did better on what measure, by how much, with both links.
- Then the differences in setup that could explain it, observed data first.
- Call out anything that makes the comparison weak: different benchmarks or task sets,
  incomplete rounds, unfinished runs, small samples.
- A table helps for three or more runs: one row per run, the same columns for each.
