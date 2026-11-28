# Single-run report

Sections, in order. Drop one only when it cannot apply, and say so under Coverage.

## Summary

Three to five sentences: what the run is, its state, the answer to the report's question
and the main reason, with the run's link.

## The run

Name, ID and link; status, with `failure.code` and `failure.message` if it failed;
spec name and `bench_id`; base model and parent checkpoint; completed against
budgeted steps, or scored, failed and unfinished rollouts; the options set away from
default.

## Reward

- Training run: reward at the start, the turning points and the latest settled step, each
  with its trajectory count; then held-out reward for completed rounds with sample counts.
- Eval run: scored, failed, cancelled and unfinished rollouts against the total, then the
  mean, labelled final or provisional.

## Completion and endings

Finished, errored and incomplete counts, termination reasons, turns and tokens against the
limits, tool failures, and for training the groups that carried signal and were used.

## Grading

The reward components with their weights and ranges, how each behaves across the sampled
trajectories, and any sign that grades are missing or constant. State the sample size.

## What the trajectories show

The trajectories you read: lowest and highest reward at the step that matters, and one per
ending that stood out. One or two sentences and an excerpt each.

## Findings

Numbered. One sentence each, then its evidence and whether it is observed or inferred.

## Likely causes and next step

Only for a diagnosis. Causes in order of support, each with evidence for, evidence against
or missing, and the cheapest experiment that would confirm it.

## Coverage

What was read, what was sampled, what failed to load, what is still running, and what the
API cannot show: optimizer metrics (loss, KL, entropy, gradient norm), unless a tool read
them.
