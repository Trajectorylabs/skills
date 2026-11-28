---
name: platform-help
description: Explain how to use Trajectory, which models can be trained, product pricing, or how to connect an assistant through MCP. Use for general getting-started, model availability, pricing, rate, and MCP connection questions before inspecting launch schemas or looking up projects, specs, runs or billing history.
metadata:
  artemis-requires: none
  artemis-user-invocable: "false"
  artemis-summary: "Explain pricing and connect an assistant"
---

# Explain the platform

Answer general product questions from the documentation below. Read the organization's resources
only when the question asks about them. The speaker's name is not a project, spec or run to find.

## Getting started

A project groups specs and their runs. A spec defines tasks, tools, the runtime and grading.
Start with an evaluation of a base model, train on the spec's training tasks, and compare the
trained checkpoints with the baseline on held-out tasks. Deploy a checkpoint when it is ready
to serve. Ask which step the person needs help with when their question is general.

## Models and pricing

Read [pricing](https://docs.trajectory.ai/pricing.md) under the `trajectory` skill with `read_skill_file` to answer general model
availability or pricing questions, such as "what models do you have for training", before
quoting models or rates. Use its trainable-model section for training availability.
It covers training, evaluation, grading, inference, sandbox compute and storage. Keep the
document's units and distinguish training, prefill, cached prefill and sampling rates.

Do not inspect launch schemas, probe the SDK or run shell commands for a general model catalog
or price list. Read schemas and model options when preparing or validating a concrete launch
configuration, not to discover the product's models or rates.

Published rates are not the organization's historical spend or its negotiated contract. Do not
read run costs or scan billing history to answer a pricing question. Read costs only when the
person asks about usage or spend; say when a contract price is not available.

## Connect an assistant through MCP

Trajectory has a hosted Streamable HTTP MCP server at `https://api.trajectory.ai/mcp`.
Add that URL as a remote MCP server in the assistant's connection settings and complete the
OAuth browser sign-in, selecting the Trajectory organization to connect.

For Codex, add the server with `codex mcp add artemis --url https://api.trajectory.ai/mcp`, then
sign in with `codex mcp login artemis`. In Claude Code, add it with
`claude mcp add --transport http artemis https://api.trajectory.ai/mcp`, then open `/mcp` and
authenticate Artemis. These are setup instructions for the person to follow, not commands to
run in your sandbox.

The server offers platform reads and run creation and cancellation tools. An MCP client calls
write tools directly, so the person should review a proposed write before their assistant runs
it. This connection is separate from MCP servers a spec gives an agent during a rollout.
