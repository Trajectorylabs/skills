---
name: run-eval
description: Run an evaluation on a Trajectory benchmark and read its result, from choosing the benchmark and model through an agreed plan and cost to the score. Use when someone wants a baseline, wants to score a base model or a trained checkpoint, or asks how a model does on a benchmark that has no eval yet.
metadata:
  artemis-requires: sandbox writes
  artemis-user-invocable: "true"
  artemis-argument-hint: "[benchmark] [model or checkpoint]"
  artemis-summary: "Evaluate a model on a benchmark"
---

# Run an eval

Builds on the `trajectory` skill: connect, find IDs and link resources the way it says.
`guides/evaluations.mdx` is the contract for options, task selection and recovery;
`pricing.mdx` has the rates.

An eval is billed for tokens, sandbox time and any model-based grading. Launch nothing
without the person's yes to a stated plan and cost. An eval does not authorize training or
deployment.

## 1. Pick the benchmark and model

- Benchmark: the `bench_id` given, or `tj benchmarks list --json` and ask which one.
  `tj benchmarks readiness BENCH_ID --json` must say ready. If not, report
  `unready_task_image_counts` and `missing_secret_names` and stop: the benchmark needs
  fixing before an eval.
- A base model: `tj evals models --json` for models and prices, then
  `tj evals options --bench-id BENCH_ID --json` for the bounds of the one you recommend.
- A trained checkpoint: `client.training.checkpoints.list(training_run_id)`, then pass its
  `checkpoint_id` as `parent_checkpoint_id` with its `base_model_slug`.
- To compare with an earlier eval, use the same `bench_id`, task selection and explicit
  options on both.

## 2. Size it

- If no task has run since the benchmark's last upload, propose a one-task diagnostic
  first (`tj benchmarks diagnose BENCH_ID --config diagnostic.json --json`), then a small
  eval to confirm a recorded grade, before the full test split.
- `val_num_tasks` caps the tasks and `val_trajectories_per_task` sets attempts per task.
- Cost: rollouts (tasks times attempts) times a rollout's tokens at the model's rates,
  plus sandbox time, plus grading if a model grades. Take tokens and duration from a
  diagnostic or an earlier eval. Without one, say the estimate is a guess and propose the
  small eval to measure it.

## 3. Confirm

State the plan in one message and wait for an explicit yes: the benchmark and `bench_id`,
the model or checkpoint, the options away from default, the number of rollouts, and the
estimated cost with what it assumes. A yes covers this plan only; a larger eval or a rerun
of failed attempts needs its own.

## 4. Launch and watch

Write the request to `eval.json` (`tj schema evals --json` gives its shape), then:

```bash
tj evals create --config eval.json --dry-run --json
tj evals create --config eval.json --json
tj evals watch EVAL_RUN_ID --timeout 300
```

Repeat `watch` until the eval is terminal; stopping it never cancels the eval. If rollouts
fail from the start, say so at once. Cancelling is the person's call
(`tj evals cancel EVAL_RUN_ID`).

## 5. Read the result

`tj evals details EVAL_RUN_ID --json`, then report it the way `explain-run` reads an eval:
counts before the mean, the mean as final only if the eval completed, and two or three
linked trajectories that show the result. Attempts that ended in `ERROR` can be rerun
alone (`client.evals.runs.resume`), which is new spend: ask first.

## Done when

The eval is `completed` and the person has its link, the counts, the mean and the
trajectories behind it; or it failed or was cancelled and they have `failure` quoted and
what was scored before it stopped.
