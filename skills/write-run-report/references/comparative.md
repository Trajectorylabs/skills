# Comparative report

Sections, in order. Every table has one row per run, the same columns for each, and the
same run order throughout.

## Summary

Which run did better on which measure, by how much, and the most likely reason, with every
run's link. If the runs are not strictly comparable, that is the first sentence.

## The runs

Per run: name and link, status, `bench_id`, base model, parent checkpoint, completed steps
or scored rollouts, created time.

## What differs

Only the config differences, laid out as the `compare-runs` skill's config diff reference
describes.

## Coverage

Per run: which data was read, how far it goes, and what failed to load. Compare only over
the range every run covers.

## Reward

- Training runs: reward at the same steps for every run, with trajectory counts, and the
  step where they part ways; then held-out reward for rounds every run completed with
  matching sample counts.
- Eval runs: scored, failed and unfinished counts, then the means, labelled final or
  provisional. Do not call a small gap over a few dozen rollouts a win.

## Completion and endings

Per run, over the same steps: share of rollouts that finished, errored and were cut off;
termination reasons; average turns and tokens; tool failures; groups with signal and used.

## Per-run findings

What is particular to each run, with numbers and at least one excerpt.

## What explains the difference

The setup differences the data supports, strongest first, each with the observation in
every run that supports it. Where the difference is in behaviour, quote a matched pair of
trajectories (same `task_id`, one per run). Name the config differences you could not
connect to any observed difference.

## Conclusions

What the comparison establishes, what it only suggests, and what would settle the rest.
