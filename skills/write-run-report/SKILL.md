---
name: write-run-report
description: Write up an analysis of one Trajectory training or eval run, or a comparison of several, as a report someone can act on and check. Use when someone asks for a report, write-up, summary or post-mortem of a run, or for the findings of an investigation in a form they can share.
metadata:
  artemis-requires: none
  artemis-user-invocable: "true"
  artemis-argument-hint: "[run IDs or names]"
  artemis-summary: "Write a shareable report on a run"
---

# Write a run report

Every tool result carries the platform URL of each resource it names: link with those.
Read only. This skill sets the standard for the report; the analysis comes from
`explain-run`, `compare-runs` or `diagnose-reward`.

Where the surface has its own length or formatting rules, such as a chat thread, those
win: send the summary section alone, in that surface's syntax, and offer the rest.

## 1. Write it

1. State the question the report answers in one sentence; without one, "what happened in
   this run, and why".
2. Put down the section headings: [references/single-run.md](references/single-run.md)
   for one run, [references/comparative.md](references/comparative.md) for several.
3. Fill each section as soon as its data is in hand, so work that is cut short leaves a
   partial report that is true.
4. Choose trajectories from the aggregates: sort by reward, step, ending and task first,
   then open the ones those numbers point at.
5. Write the summary last and put it first.

## 2. The evidence bar

- Every number comes from a tool result in this session and says what it
  measures: the run, the metric, the step, and how many trajectories it rests on. A number
  you computed says what from.
- Every run, eval and trajectory is linked the first time it appears, or named by its ID.
- A claim about counts or trends needs its numbers. A claim about what the agent did, said
  or failed to do also needs a verbatim excerpt.
- Excerpts come from more than one trajectory, from passing and failing ones, and in a
  comparison from every run.
- An excerpt is the exact text, at most five lines, in a blockquote whose first line names
  its source. Never paraphrase inside it:

  > **[trn_example · traj_example · step 3 · assistant]**
  > I'll submit the answer without running the tests.

- Observed and inferred are separate sentences, and an inference says what would confirm
  it.
- Partial data is labelled where it is used: a running run, a pending step, an incomplete
  held-out round, a provisional eval mean, a sample.

## 3. Leave out

- Numbers you cannot trace, including ones remembered from earlier in the conversation.
- Padding, and tables or plots that repeat each other.
- Options that match the default or the other runs.
- Advice the evidence does not support.
- Secrets, and private task content beyond the lines that make the point.
- Identifiers the reader did not use and cannot look anything up by.

## 4. Before you send it

Check each number against the result it came from, each link against what it opens, each
quote against the trajectory text, and the summary against the body.
