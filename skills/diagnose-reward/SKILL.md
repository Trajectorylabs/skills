---
name: diagnose-reward
description: Investigate why a Trajectory training run's reward is flat, falling, noisy or suspiciously high, and what to try next. Use when someone asks why a run is not learning, why reward will not climb or dropped, or whether a result is too good to be true, however they name the run (a run ID, a name, or the latest run on a benchmark).
metadata:
  artemis-requires: none
  artemis-user-invocable: "true"
  artemis-argument-hint: "[training run ID or name]"
  artemis-summary: "Find out why a run's reward is stuck"
---

# Diagnose reward

Every tool result carries the platform URL of each resource it names: link with those.
Read only: propose experiments, never launch, cancel or resume anything.

Do every check in order. The only early exits are in check 2. Check 6 reads trajectories
and is never skipped: the numbers show that reward is stuck, only trajectories show why.
[references/checks.md](references/checks.md) lists what a bad result looks like.

## 1. Find the run

| Given | Resolve with |
| --- | --- |
| A `trn_` ID | `get_training_run` |
| An `evr_` ID | `get_eval_run`; diagnose its `training_run_id` if it has one, otherwise use `explain-run` |
| A name, or part of one | A tool that finds runs by name, if you have one; otherwise `list_benchmarks`, then `list_training_runs` on the benchmark, matching `display_name` |
| Any other identifier | A tool that finds runs by it, if you have one; otherwise ask for the run ID or name |
| "The latest run on" a benchmark | `list_benchmarks`, then `list_training_runs`, newest first |

If several runs match and none is clearly meant, list them and ask. Name the run you
picked in the first line of the answer.

## 2. Check it is training

`get_training_run` and `get_training_progress`.

- `failed`: quote `failure.code` and `failure.message`. That is the answer; stop.
- No completed steps: the run is waiting or collecting its first batch, not flat. Run
  check 5 to see whether groups are usable, then answer.
- Fewer completed steps than `lr_warmup_steps`: say it is too early to call flat, and go on.

## 3. Config

Diff the run's `options`, base model and `parent_checkpoint_id` against the last healthy
run on the benchmark: the newest from `list_training_runs` with
`status=["succeeded"]` whose reward history rose. Follow the `compare-runs` skill's
[config diff reference](../compare-runs/references/config-diff.md) and name every
difference. With no healthy run, say the config was judged on its own.

## 4. Reward

- `get_reward_history`: name the shape (flat at zero, flat above zero,
  falling, noisy, high from the first step) from steps with `pending_count` 0. Compare the
  swing between neighbouring steps with the total change, and note `trajectory_count`.
- `get_held_out_rewards`: use only rounds with `round_completed`
  true and `completed_samples` equal to `expected_samples`. Say whether held-out reward
  follows training reward. With no completed round, generalisation is unknown.

## 5. Endings and group signal

`get_step_summaries` for every step, read with the `explain-run`
skill's [step summaries reference](../explain-run/references/step-summaries.md). Per
step, find the share of rollouts that did not end in `ENV_DONE` (`error_count`,
`incomplete_count`, `termination_reasons`) and the share of groups without signal
(`no_signal_groups` and `used_groups` against `completed_groups`). Note the step where
either share changes.

## 6. Read trajectories

Required, whatever checks 3 to 5 showed.

1. Choose the step where reward or endings changed, else the latest settled step, and one
   early step for contrast.
2. At each, take the two or three lowest-reward and the two or three highest-reward
   trajectories. Page `list_trajectories` with `training_run_id`,
   keep the ones whose `policy_step` matches, and say how many you ranked.
3. For each, `get_trajectory` for its reward components, then
   `get_trajectory_steps` from the first step: the task, what the agent did turn
   by turn, how it ended, how it was graded.

Then answer, giving the sample size:

- Components: does one sit at its maximum while the others never move, does the one that
  moves have `weight` 0, or are any missing?
- Grading: is there `ENV_DONE` with reward 0 and no components (no grade was recorded), an
  `ERROR` naming the grader or judge, or one `explanation` or one reward on every
  trajectory? `reference/troubleshooting.mdx` ("Rewards are flat at zero") tells a graded
  zero from a missing grade.
- Behaviour: did the worst fail the task, or did the environment or grader fail them? Did
  the best solve the task, or only satisfy the grader (a short answer with no tool use, an
  answer read from a file the agent should not see, full reward on a cut-off reply)?

Quote the lines that show each point, with trajectory ID, agent step and role.

## 7. Optimizer metrics

Only if you have a tool that reads the metrics the trainer logged; the read tools above do
not return them, so without one go on to check 8. List the names the run logged, then read
loss, KL, entropy, gradient norm, the learning rate applied and the reward health rates
over the whole run. [references/causes.md](references/causes.md) ("Optimizer metrics")
says what counts. A name with no data means this run did not log it, not that it is zero.

## 8. Rank the causes

Match the findings against [references/causes.md](references/causes.md). For each likely
cause give the evidence for it, the evidence against it or still missing, and the cheapest
experiment that would confirm it. Say when two causes cannot be told apart yet.

## 9. Answer

- First line: what the reward is doing and the most likely reason, with the run's link.
- The checks that found something, with their numbers and training step; one line for the
  checks that were clean.
- The ranked causes and the one experiment to run first.
- What you could not check. Unless check 7 read them, that includes the optimizer metrics
  (loss, KL, entropy, gradient norm): the API does not return them, and Trajectory can
  look them up for someone who quotes the run ID. Add any step still pending and anything
  a tool failed to return.
- Keep observed and inferred apart, and link every trajectory you quote.
