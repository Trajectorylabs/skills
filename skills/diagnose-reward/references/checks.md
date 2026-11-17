# What a bad result looks like

`guides/training.mdx` defines every option named here and `guides/termination-reasons.mdx`
every ending. Cite the option or ending with its value; do not explain it.

## Check 3: config

The `compare-runs` config diff says which differences matter. These values are suspect
even with nothing to diff against:

- `learning_rate` an order of magnitude from the model's default
  (`client.training.list_options`): too high and reward rises then collapses, too low and
  it stays flat.
- `num_steps` at or below `lr_warmup_steps`: the whole run is warm-up.
- `group_size` or `min_trajectories_per_group` at the minimum, or `temperature` near zero:
  rewards inside a group tie, so few groups carry signal.
- `train_on_truncation`, `train_on_timeout` or `train_on_error` false while check 5 shows
  most rollouts end that way: the trainer is starved.
- `train_on_all_fail`, `train_on_all_pass` or `train_on_zero_advantage` true: groups
  without signal are admitted. It does not repair a grader that returns one value.
- Turn, token or time limits below the healthy run's: rollouts are cut off before grading.
- A parent checkpoint without `load_weights_only`: the optimizer, schedule and global step
  were resumed, so the learning rate may already be decayed.
- `val_*` options that differ from their training twins: held-out reward is measured
  under other settings.

An option that is missing or null was left at its default; call it "default".

## Check 4: reward

- Flat at exactly zero, or at one constant value: suspect grading before learning.
- High from the first step: the tasks are easy for this model, or reward is earned without
  solving them. A baseline eval of the base model (`client.evals.runs.list`) tells which.
- A swing between neighbouring steps as large as the whole rise: noise. A small
  `trajectory_count` per point is noisy by construction.
- A fall that starts as warm-up ends points at the learning rate; one that starts where
  the endings change points at check 5.
- Held-out flat or falling while training reward rises: memorised tasks, a grader being
  gamed, or validation under different settings.
- Held-out rounds with few `expected_samples`, or many `failed_samples`: one task flipping
  moves the mean.

## Check 5: endings

- Mostly `TRUNCATION`, `LIMIT_REACHED` or `MAX_STEPS`: rollouts run out of turns or
  tokens. Compare `max_turns` and `max_tokens` with the run's limits.
- Mostly `TIMEOUT`: slow tools, a hung command, or a time limit set too low.
- Mostly `ERROR` or `SANDBOX_*`: the environment or grader is failing, not the model.
- `tool_failure_parse_count` rising with the step: the model's tool calls are getting
  malformed, so the policy is degrading.
- `avg_turns` or `avg_tokens` shrinking while reward rises: check 6 says whether shorter is
  better or a shortcut.

## Check 6: components and grading

- The reward is the weighted average of each component's `scaled_value`: a `weight` 0
  component moves without changing it, and one heavy component at its ceiling hides the
  rest.
- A missing secret (`client.benchmarks.retrieve_readiness`, `missing_secret_names`) stops
  a judge that needs it.
- With the SDK, `client.trajectories.list_events(tid)` shows grader output the harness
  logged.
