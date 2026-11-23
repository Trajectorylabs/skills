---
name: set-up-benchmark
description: Take a benchmark from a repository to ready on Trajectory by packaging its tasks, harness and grader, uploading it to an agent, building its runtime images and checking readiness. Use when someone wants to bring, upload, ingest or integrate a benchmark or spec, or to build the images of one that is already uploaded.
metadata:
  artemis-requires: sandbox writes
  artemis-user-invocable: "true"
  artemis-argument-hint: "[repository path or URL]"
  artemis-summary: "Bring a benchmark onto Trajectory"
---

# Set up a benchmark

Builds on the `trajectory` skill: connect, find IDs and link resources the way it says.
The docs call a benchmark a spec; a `bench_id` names one version of it.
`guides/spec.mdx` is the packaging contract.

This skill creates an agent (perhaps), a benchmark version and its images, and ends when
the benchmark is ready. It never starts a diagnostic, eval, training run or deployment:
those spend money and belong to `run-eval` and `start-training`.

## 1. Agree what will be uploaded

Before creating anything, confirm the dataset, tasks, tools and grader, skipping what the
person already decided. If the organization is not the one they named, stop.

- A benchmark in this repository: integrate it as the `trajectory` skill describes.
  `tj init --repo PATH --assistant copy --json` prints a setup prompt and starts nothing.
- No benchmark here: start from a cookbook example (`tj init --cookbook NAME`;
  `guides/cli.mdx` lists them) shaped to what the person wants the agent to do. Do not
  invent tasks.

## 2. Choose the agent

`tj agents list --json`. Reuse an agent whose name and purpose clearly match; otherwise
`tj agents create --name NAME`, named for what the agent should be good at. Ask when the
choice is not obvious.

## 3. Confirm, then upload

Say what you will create and wait for a yes: the agent (existing or new), the benchmark
name, the tasks in each split, the runtimes to build, and whether this adds a version to
an existing benchmark.

- With a manifest (`spec.model_dump_json(indent=2)`):
  `tj benchmarks push --manifest manifest.json --agent-id AGENT_ID --json`, then
  `tj operations watch OPERATION_ID`.
- From Python: `start_push` from `trajectory.lib` with `build_images=True`.

A wait that times out has not stopped the upload. Reconnect to the operation; pushing
again creates a second version.

## 4. Build images and check readiness

1. If the upload did not build, or the benchmark already existed:
   `client.benchmarks.images.build(bench_id)`.
2. Poll `client.benchmarks.images.list(bench_id)`, every page, until no image is pending
   or building.
3. `tj benchmarks readiness BENCH_ID --json`.

If `ready` is false, report `unready_task_image_counts` and `missing_secret_names` and stop
without changing the benchmark; `fix-benchmark` takes it from there.

## 5. Report, then ask what is next

Send the benchmark's link, its task count per split and its runtime status. A ready image
does not show that the harness or grader works: a one-task diagnostic does, and it needs
their go-ahead because it runs a model.

Offer only what the benchmark supports, then wait:

- A baseline eval: always, and the recommended first step.
- Training: only if the benchmark permits it and has training tasks.
- An eval of a trained model: only if a checkpoint exists.

## Done when

`ready` is true for the `bench_id` you reported, and the person has its link, task counts
and runtime status; or they have the exact blockers.
