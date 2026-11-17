# Likely causes and the cheapest next experiment

A cause needs evidence from this run; a pattern that merely fits is a hypothesis, and the
answer says which is which. Every experiment is a proposal for the person to launch.

## Flat

| Cause | Evidence | Cheapest experiment |
| --- | --- | --- |
| No grade is recorded | Check 6: `ENV_DONE`, reward 0, no components; zero from step 0 | A one-task diagnostic, then read that trajectory's components |
| The grader returns one value | Checks 4 and 6: one constant reward, identical components | The same diagnostic on a known-good and a known-bad answer |
| Rollouts do not finish | Check 5: `TRUNCATION`, `TIMEOUT` or `ERROR` dominate | A small eval of the base model with the binding limit raised |
| Groups carry no signal | Check 5: `no_signal_groups` near `completed_groups`, `used_groups` near zero | A baseline eval: `all_failed_groups` means too hard, `all_success_groups` too easy |
| Learning rate too low | Checks 3 to 5: low rate, rollouts finish, groups carry signal, still flat | A short run from the same start at the healthy run's rate |

## Falling

| Cause | Evidence | Cheapest experiment |
| --- | --- | --- |
| Updates too large | Checks 3 and 4: high `learning_rate`, fall starts as warm-up ends | A short run at a lower rate from the last checkpoint before the fall |
| The policy broke its tool use | Checks 5 and 6: parse failures climbing; malformed or repeated calls | Eval the checkpoint before the fall and the latest on the same tasks |
| The environment changed | Checks 5 and 6: `ERROR` or `SANDBOX_*` from one step on, the same failure on every task | An eval of the base model now; if it fails the same way, training is not the cause |
| Rollouts outgrew their limits | Check 5: `avg_turns` or `avg_tokens` rising to the limit, then `TRUNCATION` | An eval of the latest checkpoint with a higher limit |

## Noisy

| Cause | Evidence | Cheapest experiment |
| --- | --- | --- |
| Too few trajectories per point | Checks 3 and 4: small `trajectory_count`, `batch_size` or `group_size` | None; judge the trend over several steps |
| The grader is not deterministic | Check 6: the same answer scores differently | Grade one fixed answer several times in a diagnostic |
| Held-out rounds are small | Check 4: few `expected_samples` or many failed | An eval of one checkpoint on the full test split |

## Too good

| Cause | Evidence | Cheapest experiment |
| --- | --- | --- |
| The agent can see the answers | Check 6: the best trajectories read expected outputs, tests or grader files | Inspect the task image's files; no run needed |
| The grader rewards something else | Check 6: full reward for a short, empty or cut-off answer | Hand-check three high-reward trajectories against the task |
| Only training tasks improve | Check 4: held-out flat or falling | Eval the latest checkpoint and the base model on the test split |
| The tasks are easy | Check 4: high from the first step; `all_success_groups` dominates | A baseline eval of the base model |

## Optimizer metrics

The API does not return these. Check 7 reads them when a tool for them is available;
otherwise a cause that depends on them stays a hypothesis. Names vary with the algorithm
and backend, so list them and pick by meaning.

| Signal | What counts |
| --- | --- |
| Loss | Flat while groups carry signal: updates too small |
| KL | Near zero throughout: the policy is not moving. Spiking before reward falls: updates too large |
| Entropy, else token surprisal or mean log-probability | Near zero: the policy collapsed onto one behaviour, so rewards in a group tie |
| Gradient norm | At the clip value on every step, or near zero: the same two stories as KL |
| Learning rate applied | Against the options and the schedule in `guides/training.mdx`: still in warm-up, or decayed away, explains a flat stretch |
| Reward health rates | Zero-reward rate near 1: nothing scores, and check 6 says why. Forced-termination rate high: rollouts are cut off. Forced full-reward rate above zero: full reward for rollouts that never finished |
