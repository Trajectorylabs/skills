---
name: set-up-benchmark
description: Take a spec (formerly a benchmark) from a repository to ready on Trajectory by packaging its tasks, harness and grader, uploading it to a project, building its runtime images and checking readiness. Use when someone wants to bring, upload, ingest or integrate a spec or benchmark, or to build the images of one that is already uploaded.
metadata:
  artemis-requires: sandbox writes
  artemis-user-invocable: "true"
  artemis-argument-hint: "[repository path or URL]"
  artemis-summary: "Bring a spec onto Trajectory"
---

# Set up a spec

Builds on the `trajectory` skill: connect, find IDs and link resources the way it says.
A spec was called a benchmark, and a project an agent; `bench_id` names one version of a
spec and `agent_id` a project. The [spec guide](https://docs.trajectory.ai/guides/spec.md) is the packaging contract.

This skill creates a project (perhaps), a spec version and its images, and ends when
the spec is ready. It never starts a diagnostic, eval, training run or deployment:
those spend money and belong to `run-eval` and `start-training`.

## 1. Agree what will be uploaded

Before creating anything, confirm the dataset, tasks, tools and grader, skipping what the
person already decided. If the organization is not the one they named, stop.

- A spec or benchmark in this repository: integrate it as the `trajectory` skill describes.
  `tj init --repo PATH --assistant copy --json` prints a setup prompt and starts nothing.
- None here: start from a cookbook example (`tj init --cookbook NAME`;
  the [CLI guide](https://docs.trajectory.ai/guides/cli.md) lists them) shaped to what the person wants the agent to do. Do not
  invent tasks.

## 2. Choose the project

`tj projects list --json`. Reuse a project whose name and purpose clearly match; otherwise
`tj projects create --name NAME`, named for what the agent should be good at. Ask when the
choice is not obvious.

## 3. Confirm, then upload

Say what you will create and wait for a yes: the project (existing or new), the spec
name, the tasks in each split, the runtimes to build, and whether this adds a version to
an existing spec.

- With a manifest (`spec.model_dump_json(indent=2)`):
  `tj specs push --manifest manifest.json --agent-id PROJECT_ID --json`, then
  `tj operations watch OPERATION_ID`.
- From Python: `start_push` from `trajectory.lib` with `build_images=True`.

A wait that times out has not stopped the upload. Reconnect to the operation; pushing
again creates a second version.

## 4. Build images and check readiness

1. If the upload did not build, or the spec already existed:
   `client.specs.images.build(bench_id)`.
2. Poll `client.specs.images.list(bench_id)`, every page, until no image is pending
   or building.
3. `tj specs readiness BENCH_ID --json`.

If `ready` is false, report `unready_task_image_counts` and `missing_secret_names` and stop
without changing the spec; `fix-benchmark` takes it from there.

## 5. Report, then ask what is next

Send the spec's link, its task count per split and its runtime status. A ready image
does not show that the harness or grader works: a one-task diagnostic does, and it needs
their go-ahead because it runs a model.

Offer only what the spec supports, then wait:

- A baseline eval: always, and the recommended first step.
- Training: only if the spec permits it and has training tasks.
- An eval of a trained model: only if a checkpoint exists.

## Done when

`ready` is true for the `bench_id` you reported, and the person has its link, task counts
and runtime status; or they have the exact blockers.
