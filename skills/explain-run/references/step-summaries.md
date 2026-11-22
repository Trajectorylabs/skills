# Reading step summaries

`get_step_summaries` returns one row per training step. The fields
that explain most reward questions:

| Pattern | What it usually means | Check next |
| --- | --- | --- |
| `no_signal_groups` close to `completed_groups` | Most groups got identical rewards, so group-relative training learns little from them | `all_success_groups` (tasks too easy) versus `all_failed_groups` (too hard, or grading broken) |
| `used_groups` near zero on a settled step | The trainer consumed almost nothing at that step | `discarded_groups` and `stale_groups`; stale groups started from a policy older than the run's `max_staleness` allows |
| `pending_groups` above zero | Groups the trainer may still consume; the step is not final | `settled` |
| `error_count` high | Trajectories ended in `ERROR`; their reward is not meaningful and training skips them by default | `termination_reasons`, then a trajectory's `error` and steps |
| `incomplete_count` high | `TIMEOUT` or `TRUNCATION`: the agent ran out of turns, tokens or time | `max_turns`, `avg_tokens` against the run's limits; `train_on_truncation` and `train_on_timeout` in its options |
| `tool_failure_count` high | Tool calls that raised (`tool_failure_exec_count`) or could not be parsed (`tool_failure_parse_count`) | One affected trajectory's steps |
| `avg_turns` or `avg_tokens` moving with reward | Behaviour changed, for example shorter answers or skipped tool use | Compare a trajectory from before and after the change |

`correct_count`, `partial_count` and `incorrect_count` split finished rollouts by reward
(1, between 0 and 1, and 0). A turn is one agent step inside a trajectory; a training step
is one optimizer step. Keep the two apart when you write.

`settled` is false while `used_groups` and `discarded_groups` can still change. Do not
call a recent, unsettled step's counts final.
