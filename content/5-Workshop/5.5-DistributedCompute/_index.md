---
title: "Phase 3 - Distributed Compute"
date: 2026-07-30
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
includeInReport: true
---

### Objective

Turn the local, single-run backtest engine from Phase 2 into a distributed job that evaluates an entire parameter grid in parallel: one **Worker Lambda invocation per parameter combination**, fanned out by a **Step Functions Distributed Map**, with results ranked and persisted by an **Aggregator Lambda**.

### Architecture

![Phase 3 - Distributed Compute architecture](/images/5-Workshop/phase3_distributed_compute.png)

### State machine - `infra/stacks/compute_stack.py`

```python
distributed_map = sfn.DistributedMap(
    self, "BacktestDistributedMap",
    max_concurrency=50,
    items_path="$.items",
    result_writer=sfn.ResultWriter(bucket=data_bucket, prefix="job-results/"),
    item_selector={
        "params": sfn.JsonPath.string_at("$$.Map.Item.Value"),
        "bucket": sfn.JsonPath.string_at("$.bucket"),
        "symbol": sfn.JsonPath.string_at("$.symbol"),
        ...
    },
)
distributed_map.item_processor(worker_task)

distributed_map.add_catch(mark_failed_task, errors=["States.ALL"], result_path="$.error")
aggregate_task.add_catch(mark_failed_task, errors=["States.ALL"], result_path="$.error")

definition = distributed_map.next(aggregate_task)
self.state_machine = sfn.StateMachine(
    self, "BacktestStateMachine",
    definition_body=sfn.DefinitionBody.from_chainable(definition),
    timeout=Duration.minutes(20),
)
```

The `add_catch` wiring means a job that fails partway through is marked `FAILED` in DynamoDB with the error recorded, instead of hanging in `RUNNING` forever.

### Orchestrator - `services/orchestrator/handler.py`

Turns a symbol/strategy/date-range/param-grid request into a Distributed Map item list and starts an execution:

```python
def handler(event, context):
    job_id = str(uuid.uuid4())
    param_sets = _expand_grid(event["param_grid"])   # cartesian product

    jobs_table.put_item(Item={"jobId": job_id, "status": "RUNNING", ...})
    sfn_client.start_execution(
        stateMachineArn=STATE_MACHINE_ARN,
        name=job_id,
        input=json.dumps({"jobId": job_id, "items": param_sets, ...}),
    )
    return {"jobId": job_id, "executionArn": ..., "paramCount": len(param_sets)}
```

### Deploy

```bash
cd infra
cdk deploy ServerlessFinance-Compute --require-approval never
```

```
 ServerlessFinance-Compute
Outputs:
ServerlessFinance-Compute.OrchestratorFunctionName = ServerlessFinance-Compute-OrchestratorFunctionA735-ArVp2lZbvqEd
ServerlessFinance-Compute.StateMachineArn = arn:aws:states:ap-southeast-1:752643409325:stateMachine:BacktestStateMachineDCBCAFEE-k1Zn1DbvZfvV
ServerlessFinance-Compute.JobsTableName = ServerlessFinance-Compute-BacktestJobsTable1A6FFBD6-1K2RJ06PJNHP0
ServerlessFinance-Compute.ResultsTableName = ServerlessFinance-Compute-BacktestResultsTable00881B48-6E2PB6IWGFPW
```

### Verify - run a real job through Step Functions

{{% notice note %}}
Kept to a 4-combination grid here rather than 50–100, this account's default Lambda concurrency quota is 10 (see Phase 6), and a larger grid intermittently throttles the Distributed Map. 16-combination jobs *were* run successfully during Phase 6's benchmarking (sequentially, bypassing the Map).
{{% /notice %}}

```bash
$ aws lambda invoke \
  --function-name ServerlessFinance-Compute-OrchestratorFunctionA735-ArVp2lZbvqEd \
  --payload '{
    "symbol": "BTCUSDT", "interval": "1h", "strategy": "sma_crossover",
    "start_date": "2025-01-01", "end_date": "2025-06-30",
    "param_grid": {"fast_period": [5, 10], "slow_period": [30, 50]}
  }' \
  --cli-binary-format raw-in-base64-out out.json

{"jobId": "7fc7eb24-5bf3-41cd-9933-cbb2104bc55a", "executionArn": "arn:aws:states:...:execution:BacktestStateMachineDCBCAFEE-k1Zn1DbvZfvV:7fc7eb24-...", "paramCount": 4}
```

```bash
$ aws stepfunctions describe-execution --execution-arn <executionArn>
```

```json
{
    "status": "SUCCEEDED",
    "startDate": "2026-07-29T22:29:08.078+07:00",
    "stopDate": "2026-07-29T22:29:31.877+07:00",
    "output": "{\"jobId\": \"7fc7eb24-...\", \"status\": \"SUCCEEDED\", \"resultCount\": 4}"
}
```

```bash
$ aws dynamodb query --table-name ServerlessFinance-Compute-BacktestResultsTable00881B48-... \
  --key-condition-expression "jobId = :j" --expression-attribute-values '{":j":{"S":"7fc7eb24-..."}}'
```

| rank | fast/slow | total_return | sharpe_ratio | max_drawdown | win_rate | num_trades |
|---|---|---|---|---|---|---|
| 0 | 5/30 | -13.27% | -0.664 | -25.42% | 31.4% | 105 |
| 1 | 10/30 | -16.51% | -0.923 | -29.91% | 31.0% | 87 |
| 2 | 10/50 | -18.33% | -1.058 | -34.11% | 33.9% | 59 |
| 3 | 5/50 | -24.29% | -1.459 | -36.85% | 31.0% | 84 |

Results are already sorted by the Aggregator (rank 0 = best Sharpe). SMA crossover on BTCUSDT H1-2025 loses money at every tested parameter combination, which is useful (but not pretty) result: it means this particular grid isn't a strategy worth paper-trading in Phase 5 without further tuning.

Continue to [Phase 4 - API Layer](../5.6-APILayer/).
