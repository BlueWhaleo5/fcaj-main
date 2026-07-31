---
title: "Phase 3 - Compute Phân tán"
date: 2026-07-30
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
includeInReport: true
---

### Mục tiêu

Biến engine backtest cục bộ, chạy một lần từ Phase 2 thành một job phân tán đánh giá toàn bộ lưới tham số song song: **một lượt gọi Worker Lambda cho mỗi tổ hợp tham số**, phân tán bởi **Step Functions Distributed Map**, kết quả được xếp hạng và lưu bởi **Aggregator Lambda**.

### Kiến trúc

![Kiến trúc Phase 3 - Distributed Compute](/images/5-Workshop/phase3_distributed_compute.png)

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

Cấu hình `add_catch` nghĩa là một job lỗi giữa chừng sẽ được đánh dấu `FAILED` trong DynamoDB kèm lỗi ghi lại, thay vì treo mãi ở trạng thái `RUNNING`.

### Orchestrator - `services/orchestrator/handler.py`

Biến một request (symbol/strategy/khoảng ngày/lưới tham số) thành danh sách item cho Distributed Map và khởi chạy một execution:

```python
def handler(event, context):
    job_id = str(uuid.uuid4())
    param_sets = _expand_grid(event["param_grid"])   # tích Descartes

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

### Kiểm chứng - chạy job thật qua Step Functions

{{% notice note %}}
Ở đây dùng lưới 4 tổ hợp thay vì 50–100, quota concurrency Lambda mặc định của account này là 10 (xem Phase 6), lưới lớn hơn thỉnh thoảng làm Distributed Map bị throttle. Job 16 tổ hợp *đã* chạy thành công thật trong lúc benchmark Phase 6 (chạy tuần tự, không qua Map).
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
| 0 | 5/30 | -13,27% | -0,664 | -25,42% | 31,4% | 105 |
| 1 | 10/30 | -16,51% | -0,923 | -29,91% | 31,0% | 87 |
| 2 | 10/50 | -18,33% | -1,058 | -34,11% | 33,9% | 59 |
| 3 | 5/50 | -24,29% | -1,459 | -36,85% | 31,0% | 84 |

Kết quả đã được Aggregator sắp xếp sẵn (rank 0 = Sharpe tốt nhất). SMA crossover trên BTCUSDT khung H1 năm 2025 lỗ ở mọi tổ hợp tham số đã test, đây là một kết quả có ích (dù không đẹp): nghĩa là lưới tham số này chưa phải chiến lược đáng paper-trade ở Phase 5 nếu không tinh chỉnh thêm.

Tiếp tục tới [Phase 4 - Lớp API](../5.6-APILayer/).
