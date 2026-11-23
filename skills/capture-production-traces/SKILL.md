---
name: capture-production-traces
description: Connect an existing application's agent to Trajectory so its real conversations are recorded as production trajectories on an agent. Use when someone wants to capture, log or collect production traces or live traffic, or to attach their application's model calls to a Trajectory agent.
metadata:
  artemis-requires: sandbox
  artemis-user-invocable: "true"
  artemis-argument-hint: "[agent name]"
  artemis-summary: "Record your app's live agent conversations"
---

# Capture production traces

Builds on the `trajectory` skill: connect, find IDs and link resources the way it says.
Production traces must be enabled for the organization. `guides/production-traces.mdx` is
the contract for the integration (`tj docs --topic capture --json` locates it); read it
first.

This skill changes the person's application and sends one real conversation to
Trajectory. It needs no benchmark, training run or deployment: create none.

## 1. Check the ground

- The agent the traces belong to: the `agent_id` given, or `tj agents list --json` and
  ask. Create one only if asked.
- If a call says capture is not enabled, stop and tell the person to ask Trajectory for
  access.

## 2. Integrate

Find where the application calls its model, where a conversation starts and ends, and what
tells it the outcome. `tj init --template connect --assistant copy --json` prints a setup
prompt and starts nothing. Then follow the guide:

- One trajectory per conversation, attached with `agent_id`, with an idempotency key from
  the application's own request ID.
- That trajectory's ID on every model call in the conversation.
- A reward only where the application already knows the outcome. Do not invent a grader.
- Completion with the ending that happened (`guides/termination-reasons.mdx`).

Keep the application's model, deployment, prompts, tools, retries and error handling. If
capture would mean switching models or providers, stop and put that choice to the person.

## 3. Confirm before sending anything

Captured conversations are stored by Trajectory, and model calls routed through it are
billed per token. Before the first real call, wait for a yes to:

- the conversation used to verify, chosen by the person. Do not pick a private one
  yourself or invent a model call;
- the fields that leave the application.

## 4. Verify one conversation

Run it through the integrated path, then read it back:
`client.trajectories.retrieve(trajectory_id)` and
`client.trajectories.steps.list(trajectory_id)`. Every turn must be there, in order, with
the right ending; a trajectory that is empty or never completed is not verified. Link it
with `PlatformLinks.production(agent_id, trajectory_id=...)`.

## Done when

One real conversation, chosen by the person, is stored on the agent as a completed
production trajectory, and they have its link and the list of files you changed. If
capture is not enabled or the model cannot be kept, they know that and nothing was sent.
