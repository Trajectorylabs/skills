---
name: download-model
description: Download a trained Trajectory checkpoint's weights, either its LoRA adapter or the full model with the adapter merged in, to a folder the person chooses. Use when someone wants to download, export or take a checkpoint or trained model off the platform, or to run it on their own serving stack.
metadata:
  artemis-requires: sandbox
  artemis-user-invocable: "true"
  artemis-argument-hint: "[checkpoint or run] [adapter or merged]"
  artemis-summary: "Download a trained checkpoint's weights"
---

# Download a model

Builds on the `trajectory` skill: connect, find IDs and link resources the way it says.
Checkpoint downloads must be enabled for the organization, and merged models separately.
The [checkpoint download guide](https://docs.trajectory.ai/guides/checkpoint-downloads.md) is the contract: supported models, precisions, sizes,
arguments and errors. There is no `tj` command for this; use the SDK.

This skill writes files to the person's disk. Do not start a training run, eval or
deployment.

## 1. Pin down the checkpoint

The `checkpoint_id` given, or `client.rl.checkpoints.list(training_run_id)`: show the
steps and ask which, unless they said "latest" (the highest `step_index`). Do not pick one
by reward without saying so; held-out reward by step
(`client.rl.rewards.list_held_out_rewards`) is the evidence for "best".

## 2. Choose the format

- The LoRA adapter, to keep training or serve with a stack that loads adapters:
  `download_checkpoint`.
- The merged model, one Hugging Face folder: `download_merged_export`.
  `client.beta.checkpoint_exports.list(checkpoint_id).available_merge_precisions` lists the
  precisions this checkpoint supports with their sizes and limits. If it is empty, the
  platform cannot merge this checkpoint; the guide says how to merge it yourself.

## 3. Confirm

Ask where to save it and, for a merged model, which precision. Check the location has room
for the reported size. Say that preparing a merged model can take over an hour and that a
queued export cannot be cancelled.

## 4. Download

Both helpers are in `trajectory.lib.beta`.

- Adapter: `download_checkpoint(client, checkpoint_id, output_dir)`. A failed call leaves
  nothing behind; call it again.
- Merged model:
  1. `client.beta.checkpoint_exports.create` with
     `body={"export_type": "merged_model", "precision": precision}`. Tell the person it is
     queued, with the `export_id`. Calling it again reuses the same export.
  2. Poll `client.beta.checkpoint_exports.retrieve(export_id)` about once a minute until
     `state` is `ready` or `failed`. On `failed`, report the failure and stop.
  3. `download_merged_export(client, export_id, output_dir)`. If it stops partway, call it
     again; finished files are kept.

403 `beta_not_enabled` means the organization has no access: stop and tell the person to
ask Trajectory. On 409, check `available_merge_precisions` and offer the adapter. Ask
before passing `overwrite=True`.

## Done when

The folder exists at the path you reported with the complete download, and the person has
its path, size, checkpoint and step, and for a merged model the precision and the advice to
evaluate it on their own serving stack, since merging can cost accuracy. If the export
failed or access was refused, they have the error quoted and what to ask for.
