---
name: fix-benchmark
description: Work out why a Trajectory benchmark cannot launch or does not grade (failed image builds, tasks missing a run command or runtime, missing secrets, a failed diagnostic, grader errors, readiness that stays false) and fix it in the benchmark's source. Use when a benchmark is not ready, an upload or build failed, or its runs record errors or no grades.
---

# Fix a benchmark

Builds on the `trajectory` skill: connect, find IDs and link resources the way it says.
The docs call a benchmark a spec; a `bench_id` names one version of it.

A benchmark is fixed in its source and uploaded again as a new version, never patched
around with a second, partial benchmark. Do not start an eval, training run or deployment.

## 1. Find what is wrong

All reads. Quote the message and name the task or runtime before proposing a change.

- Readiness: `tj benchmarks readiness BENCH_ID --json` gives `ready`,
  `unready_task_image_counts` and `missing_secret_names`.
- Tasks needing setup: `client.benchmarks.tasks.list(bench_id, environment="setup_needed")`,
  every page, shows each task's `run_command` and `runtime_id`.
- Images: `client.benchmarks.images.list(bench_id)`, every page, shows each task's sandbox
  provider and `build_status`.
- A failed upload: `get_operation(client, operation_id)` from `trajectory.lib`, then
  `pages("failure")`.
- A task that ran: `tj diagnostics results DIAGNOSTIC_ID --json` gives each task's
  `termination_reason`, `reward` and `failure`.

## 2. Decide the change

| What you see | Change |
| --- | --- |
| A task with no `run_command` | Add the command that runs the agent for that task |
| No `runtime_id`, or an image with no `build_status` | Set or register the task's runtime |
| An unsupported sandbox provider | Move the task's runtime to a supported provider |
| A failed image | Fix the Dockerfile or its context. If the source was not at fault, rebuild with `client.benchmarks.images.build(bench_id)`; a build that fails twice the same way needs a source change |
| `missing_secret_names` | The person adds the secret under Settings. Nothing to upload |
| An error message, or a failure when a task runs | Search `reference/troubleshooting.mdx` for the exact text |
| `TRUNCATION` or `TIMEOUT` on tasks the source agent finishes | A harness limit tighter than the run's options is binding. Keep harness limits at or above the run's |
| Rollouts that sit until the timeout after the agent answered | The harness waits on a background process. End a timed-out command's whole process tree and exit after the final answer |

Keep tasks, splits, task IDs and scoring unchanged unless the failure is in one of them.

## 3. Confirm, then upload a new version

Say what changed and that the upload returns a new `bench_id`, then wait for a yes. Push
to the benchmark's own agent (`client.benchmarks.retrieve(bench_id).agent_id`) under the
same name with `tj benchmarks push --manifest manifest.json --agent-id AGENT_ID --json` or
`start_push`. Then build and check readiness as in `set-up-benchmark` step 4.

## 4. Prove a run-time fix

If the failure was in the harness, the grader or an ending, readiness proves nothing. A
diagnostic runs a model in a sandbox and costs money: name the task, model and options,
and wait for a yes.

`tj benchmarks diagnose BENCH_ID --config diagnostic.json --json`, then
`tj diagnostics watch DIAGNOSTIC_ID` and `tj diagnostics results DIAGNOSTIC_ID --json`.
Check the affected task's ending, reward and reward components
(`client.trajectories.retrieve` with `include_reward_components=True`). A graded zero is a
valid result; a missing grade is not fixed.

## Done when

`ready` is true for the new `bench_id`, reported with its link, and for a run-time failure
the affected task ended as it should with a recorded grade. Otherwise say what is still
blocked.
