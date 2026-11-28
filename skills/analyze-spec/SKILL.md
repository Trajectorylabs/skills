---
name: analyze-spec
description: Analyze a spec or benchmark's task coverage, splits, grading and verifiable gaps or risks. Use when asked to analyze a spec, summarize its tasks and grader, or assess its quality without publishing an audit.
metadata:
  artemis-requires: sandbox
  artemis-user-invocable: "false"
  artemis-summary: "Analyze task coverage and grading"
---

# Analyze a spec

Read only. Analyze the requested spec; never start runs, change tasks or publish findings.
Get its verified ID and link with `get_spec`, or find it with `list_specs` first. A spec
ID names one version: keep evidence from other versions separate.

## 1. Collect coverage and run inventory together

Use one script for the full task inventory, category-by-split counts, runtime/build
status, repeated labels, cached readiness and the spec's training/eval inventories.
Replace `bm_verified` below with the verified ID. Execute the whole block together;
do not return to the model between fetching tasks and counting their categories.

```python
import json
from collections import Counter
from concurrent.futures import ThreadPoolExecutor
from pathlib import Path
from trajectory import Client

SPEC_ID = "bm_verified"
client = Client()
with ThreadPoolExecutor(max_workers=8) as pool:
    details_job = pool.submit(
        client.specs.details.retrieve, SPEC_ID,
        include_tasks=True, include_tools=True, include_runtimes=True,
    )
    readiness_job = pool.submit(
        client.specs.retrieve_readiness, SPEC_ID, extra_query={"refresh": False},
    )
    training_job = pool.submit(client.rl.runs.list, SPEC_ID, limit=20)
    eval_job = pool.submit(client.evals.runs.list, SPEC_ID, limit=20)
    completed_job = pool.submit(
        client.evals.runs.list, SPEC_ID, status=["completed"], limit=1,
    )
    details = details_job.result().model_dump(mode="json")
    readiness = readiness_job.result().model_dump(mode="json")
    training_page, eval_page = training_job.result(), eval_job.result()
    training = [run.model_dump(mode="json") for run in training_page.items]
    evals = [run.model_dump(mode="json") for run in eval_page.items]
    completed = [run.model_dump(mode="json") for run in completed_job.result().items]
tasks = details["tasks"]
category_splits = Counter(
    (category["task_category_id"], task["split"])
    for task in tasks for category in task["task_categories"] or []
)
summary = {
    "tasks_read": len(tasks), "tasks_declared": details["task_count"],
    "splits": dict(Counter(task["split"] for task in tasks)),
    "category_splits": [
        {"category": category, "split": split, "count": count}
        for (category, split), count in category_splits.items()
    ],
    "uncategorized_tasks": sum(not task["task_categories"] for task in tasks),
    "tasks_with_visible_input_messages": sum(
        bool(task["input_message"]) and task["input_message"] != "[hidden for this key]"
        for task in tasks
    ),
    "tools_read": len(details["tools"] or []),
    "runtimes_read": len(details["runtimes"] or []),
    "build_statuses": dict(Counter(task["build_status"] for task in tasks)),
    "runtimes": dict(Counter(task["runtime_id"] for task in tasks)),
    "duplicate_labels": [
        label for label, count in Counter(task["label"] for task in tasks).items()
        if label is not None and count > 1
    ],
    "readiness": readiness,
    "training_runs_read": len(training), "training_runs_have_more": training_page.has_more,
    "eval_runs_read": len(evals), "eval_runs_have_more": eval_page.has_more,
    "latest_completed_eval": next(({
        field: run[field] for field in (
            "eval_run_id", "status", "reward_mean", "total_rollouts", "training_run_id", "step",
        )
    } for run in completed), None),
}
Path("spec_evidence.json").write_text(json.dumps({
    "details": details, "training_runs": training, "eval_runs": evals,
    "summary": summary,
}))
print(json.dumps(summary))
```

Run inventories above are the newest page only, not an all-history count; `has_more`
says whether older runs exist. The filtered query finds the newest completed eval without
scanning its history. Full task coverage does not require reading every old run.
SDK page iteration follows every page; `limit` sets page size, not a total, while `.items`
is one page. Do not combine iteration with a cursor loop. Check tasks read against tasks
declared before claiming full coverage.
Missing or hidden task messages, commands, tools or runtimes are an access gap, not proof
that the spec has no prompt, harness or grader. Readiness checks image preparation, not
grading correctness. This batch already reads the same fields as `get_spec_details`;
rereading that tool cannot reveal hidden configuration. Record the access gap and use
the saved evidence for further local counts and comparisons.

## 2. Collect grading evidence in a batch

Use a supplied evaluation run, otherwise the latest completed eval on this spec, and state the
choice. With no recorded rollouts, report grading as unverified rather than launching an
eval. Replace `evr_verified` below. Collect the full rollout inventory and scored rewards
together, then enrich rows with reward components in parallel without replacing their
list metadata with the detail response.

```python
import json
from collections import Counter
from concurrent.futures import ThreadPoolExecutor
from pathlib import Path
from trajectory import Client

EVAL_RUN_ID = "evr_verified"
client = Client()
spec = json.loads(Path("spec_evidence.json").read_text())
run = client.evals.runs.retrieve(EVAL_RUN_ID)
if run.bench_id != spec["details"]["bench_id"]:
    raise ValueError("The eval is not on the requested spec version")
with ThreadPoolExecutor(max_workers=8) as pool:
    inventory_job = pool.submit(
        lambda: list(client.trajectories.list(eval_run_id=EVAL_RUN_ID))
    )
    rewards_job = pool.submit(
        lambda: list(client.evals.rewards.list_trajectory_rewards(EVAL_RUN_ID))
    )
    inventory = inventory_job.result()
    scored = {
        row.trajectory_id: row.model_dump(mode="json") for row in rewards_job.result()
    }
    def enrich(row):
        detail = client.trajectories.retrieve(
            row.trajectory_id, include_steps=False, include_reward_components=True,
        )
        return {
            **row.model_dump(mode="json"),
            "scored_reward": scored.get(row.trajectory_id),
            "reward_components": detail.model_dump(mode="json")["reward_components"],
        }
    rows = list(pool.map(enrich, inventory))
Path("grading_evidence.json").write_text(json.dumps(rows))
raw_rewards = [row["reward"] for row in rows if row["reward"] is not None]
scored_rewards = [row["reward"] for row in scored.values()]
print(json.dumps({
    "eval_run_id": EVAL_RUN_ID, "rollouts_read": len(rows),
    "scored_rewards_read": len(scored),
    "terminations": dict(Counter(row["termination_reason"] for row in rows)),
    "raw_reward": {
        "count": len(raw_rewards),
        "mean": sum(raw_rewards) / len(raw_rewards) if raw_rewards else None,
    },
    "eval_scored_reward": {
        "count": len(scored_rewards),
        "mean": sum(scored_rewards) / len(scored_rewards) if scored_rewards else None,
    },
    "component_names": dict(Counter(
        component["name"] for row in rows for component in row["reward_components"] or []
    )),
}))
```

The scored-rewards endpoint omits unscored and errored/cancelled attempts; it does not
replace the full rollout inventory. It can score a truncated trajectory as zero even
when its raw logged reward differs. Use its effective rewards for eval means and its
task IDs as authoritative scored-task evidence.
A null field in trajectory detail does not erase a task ID or run association returned
by listing. Verify apparent missing task mappings with scored rewards before calling
the eval uncomparable.

Group reward explanations and outcomes locally from the saved rows. In one further
script, retrieve `client.trajectories.steps.list(trajectory_id)` for the suspected cases
and matching successful controls together. Follow every page for each inspected case.
Inspect task messages and grader explanations together: a low reward or a repeated
answer is not by itself a grading defect. Save complete steps locally and print the
specific evidence needed to verify each claim. Do not make a separate model round trip
for each count, component name, example or task.

## 3. Report coverage and verified risks

- State task/split/category counts, runtime readiness and which run supplied grading
  evidence. Distinguish overall rollout inventory from the cases whose steps you read.
- Explain the scoring rule from visible configuration or recorded grader evidence.
  Separate the reward used for scoring from diagnostic metrics.
- Verify a suspected cause across all candidate tasks before reporting its affected
  count. A sample establishes an example, not a full-spec count. Report unreadable or
  uninspected evidence explicitly; never silently cap a scan and call it complete.
- If a checkpoint trained on another spec, verify its training run's spec and compare
  the requested split. Matching labels alone cannot prove prompt-level overlap or its
  absence. Batch any required cross-version reads and comparisons.
- Give each risk its evidence, affected coverage and smallest suggested correction.
  Keep inference separate from observed behavior. This analysis never publishes findings;
  publishing an audit is a separate request.
