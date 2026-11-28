# Read independent trajectories together

Use this recipe with a sandbox after listing the trajectories to inspect. Save the
selected list rows as `trajectory_inventory.json`, a JSON array retaining their
`trajectory_id`, task/run association, policy step, status, termination and reward fields.
Include all candidates required by the claim and successful controls; a selected sample
does not establish full-run coverage.

Execute the whole block together. At most eight workers perform blocking SDK reads.
Each trajectory's steps follow all pages in API order; results retain the input order
even when another trajectory finishes first. Keep list metadata alongside the detail
response: a null detail field must not erase a canonical association from the inventory.
API errors abort the script instead of silently dropping unreadable evidence.

```python
import json
from concurrent.futures import ThreadPoolExecutor
from pathlib import Path
from trajectory import Client

inventory = json.loads(Path("trajectory_inventory.json").read_text())
with Client() as client:
    def read_trajectory(row):
        trajectory_id = row["trajectory_id"]
        detail = client.trajectories.retrieve(
            trajectory_id, include_steps=False, include_reward_components=True,
        )
        steps = [
            step.model_dump(mode="json")
            for step in client.trajectories.steps.list(
                trajectory_id, include_decisions=True, limit=50,
            )
        ]
        return {"inventory": row, "detail": detail.model_dump(mode="json"), "steps": steps}
    with ThreadPoolExecutor(max_workers=8) as pool:
        evidence = list(pool.map(read_trajectory, inventory))
Path("trajectory_evidence.json").write_text(json.dumps(evidence))
print(json.dumps({
    "trajectories_read": len(evidence),
    "steps_read": sum(len(row["steps"]) for row in evidence),
}))
```

Reuse the saved evidence for local counts and signature checks. Inspect cumulative
message histories and explicit compaction before concluding that earlier messages were
lost. Print only the relevant step indices and excerpts into the model context; retain
the full steps in the file for verification. Do not print every cumulative history or
return to the model between reading each trajectory.
