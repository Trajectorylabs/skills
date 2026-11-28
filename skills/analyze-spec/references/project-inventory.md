# Full project run inventory

Use this recipe only with a sandbox and a verified project ID from `list_projects`.
Read both project run collections directly: listing every spec and then each spec's runs
adds unnecessary requests. The run finders return bounded newest results and have no
project filter; they cannot establish full project coverage.

Replace `agt_verified` and execute the complete block once. SDK iteration follows every
page; `limit` is a page size, not a total cap. The two blocking reads overlap with at most
eight workers, while each collection keeps its API order. API errors abort the script;
do not report a partial inventory as complete.

```python
import json
from concurrent.futures import ThreadPoolExecutor
from pathlib import Path
from trajectory import Client

PROJECT_ID = "agt_verified"
with Client() as client, ThreadPoolExecutor(max_workers=8) as pool:
    training_job = pool.submit(lambda: [
        run.model_dump(mode="json")
        for run in client.projects.list_training_runs(PROJECT_ID, limit=200)
    ])
    eval_job = pool.submit(lambda: [
        run.model_dump(mode="json")
        for run in client.projects.list_evaluation_runs(PROJECT_ID, limit=200)
    ])
    training, evals = training_job.result(), eval_job.result()
Path("project_inventory.json").write_text(json.dumps({
    "project_id": PROJECT_ID, "training_runs": training, "eval_runs": evals,
}))
runs = training + evals
print(json.dumps({
    "project_id": PROJECT_ID, "training_runs_read": len(training),
    "eval_runs_read": len(evals),
    "specs_with_runs": len({run["bench_id"] for run in runs if run["bench_id"] is not None}),
    "runs_without_spec": sum(run["bench_id"] is None for run in runs),
}))
```

The saved file retains every returned run field. Reuse it for counts, options and run
selection rather than fetching the same inventory again. `specs_with_runs` counts only
spec versions represented by runs, not all specs in the project. Keep comparisons on
different spec versions separate, and label this as a read snapshot if runs are changing.
