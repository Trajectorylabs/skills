---
name: start-training
description: Size, launch and watch a first training run on a Trajectory spec (formerly a benchmark), with an agreed budget and a rule for when to stop. Use when someone wants to train or fine-tune a model on a spec or benchmark, start a smoke training run, or resume training from a checkpoint.
metadata:
  artemis-requires: sandbox writes
  artemis-user-invocable: "true"
  artemis-argument-hint: "[spec] [base model]"
  artemis-summary: "Plan and launch a training run"
---

# Start training

Builds on the `trajectory` skill: connect, find IDs and link resources the way it says.
The [training guide](https://docs.trajectory.ai/guides/training.md) defines every option and how each affects spend; [pricing](https://docs.trajectory.ai/pricing.md) has
the rates.

A training run is billed for sampled and trained tokens and for sandbox time, and nothing
in its options caps its total trajectories. Launch nothing without the person's yes to a
stated plan, budget and stop rule. Never deploy a model from this skill.

## 1. Check the ground

- The spec: its ID (`bench_id`) given, or `tj specs list --json` and ask which one.
  `tj specs readiness BENCH_ID --json` must say ready.
- It must permit training and have training tasks; otherwise say so and stop.
- A baseline: `tj evals list --bench-id BENCH_ID --json`. Without a completed eval of the
  base model there is nothing to compare a checkpoint with and no proof the grader records
  grades. Recommend `run-eval` first; if they decline, say what will be unknown.
- The model: `tj training models --json` for prices, then
  `tj training options --bench-id BENCH_ID --json` for bounds and defaults. Use the
  baseline's model unless told otherwise.

## 2. Size a first run

A first run shows that steps complete, groups carry signal and reward is recorded. It does
not establish improvement.

- Set `num_steps` (enough to get past `lr_warmup_steps`), `batch_size`, `group_size` and
  the validation cadence (`val_interval_steps`, `val_num_tasks`). Keep
  `max_concurrent_trajectories` above `batch_size` times `group_size`. Keep the per-rollout
  limits the baseline used, and leave the learning rate, schedule, filters and sampling at
  default unless you can state a reason.
- Trajectories: `num_steps` times `batch_size` times `group_size`, plus rollouts collected
  but discarded (more when the baseline passes nearly all or nearly none of the tasks),
  plus validation rounds times `val_num_tasks` times `val_trajectories_per_task`. Give a
  range.
- Cost: tokens per rollout from the baseline at the model's prefill, sample and train
  rates, plus sandbox time, plus grading if a model grades. With no baseline, say the
  estimate is a guess.
- Resuming: pass `parent_checkpoint_id`, and say whether the plan resumes the optimizer
  and schedule (the default) or sets `load_weights_only`.

## 3. Confirm

State the plan in one message and wait for an explicit yes: the spec and `bench_id`,
the model, the options away from default, the expected trajectories and cost with what
they assume, and the stop rule. A workable stop rule: "if it has started N rollouts and
completed no step, I will come back and recommend cancelling", with N a small multiple of
`batch_size` times `group_size`. A yes covers this run only.

## 4. Launch and watch

Write the request to `training.json` (`tj schema training --json` gives its shape), then:

```bash
tj training create --config training.json --dry-run
tj training create --config training.json --json
tj training watch TRAINING_RUN_ID --timeout 300
```

Repeat `watch`; stopping it never cancels the run. Between waits read
`tj training details TRAINING_RUN_ID --json` (progress, reward history, checkpoints) and
`client.rl.runs.list_step_summaries(run_id)`. Healthy is completed steps rising,
`used_groups` above zero on settled steps, and most rollouts ending in `ENV_DONE`.

## 5. Know when to stop

Come back with numbers and recommend cancelling when:

- the run passed its agreed budget without completing a step;
- settled steps keep `used_groups` at zero, or `no_signal_groups` equal to
  `completed_groups`;
- most rollouts end in `ERROR`, `TIMEOUT` or `TRUNCATION`;
- reward stays flat well past warm-up (`diagnose-reward` says why).

On their yes, `tj training cancel TRAINING_RUN_ID`, then watch until the status is
`cancelled`. Fix the cause before proposing another run.

## Done when

The run is `succeeded` and the person has its link, steps, training reward at the start
and end, held-out reward for completed rounds, and its checkpoints
(`client.rl.checkpoints.list`), with an offer to eval one through `run-eval`. If it
failed or was cancelled: `failure` quoted or the reason it was stopped, and what it spent.
