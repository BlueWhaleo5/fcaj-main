---
title: "Phase 4 - Lớp API"
date: 2026-07-30
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
includeInReport: true
---

### Mục tiêu

Đặt một REST API trước Orchestrator (tạo job backtest) và một Lambda chỉ đọc nhỏ (trạng thái job, kết quả xếp hạng), bảo vệ bằng API key.

### Hạ tầng - `infra/stacks/api_stack.py`

```python
api = apigw.RestApi(
    self, "BacktestApi",
    rest_api_name="ServerlessFinance Backtest API",
    deploy_options=apigw.StageOptions(stage_name="prod", throttling_rate_limit=10, throttling_burst_limit=20),
)

api_key = api.add_api_key("DefaultApiKey")
usage_plan = api.add_usage_plan(
    "DefaultUsagePlan",
    throttle=apigw.ThrottleSettings(rate_limit=10, burst_limit=20),
    api_stages=[apigw.UsagePlanPerApiStage(api=api, stage=api.deployment_stage)],
)
usage_plan.add_api_key(api_key)

backtests = api.root.add_resource("backtests")
backtests.add_method("POST", apigw.LambdaIntegration(orchestrator_fn, proxy=True), api_key_required=True)

job_resource = backtests.add_resource("{jobId}")
job_resource.add_method("GET", apigw.LambdaIntegration(self.api_fn, proxy=True), api_key_required=True)

results_resource = job_resource.add_resource("results")
results_resource.add_method("GET", apigw.LambdaIntegration(self.api_fn, proxy=True), api_key_required=True)
```

### Deploy

```bash
cd infra
cdk deploy ServerlessFinance-Api --require-approval never
```

```
 ServerlessFinance-Api
Outputs:
ServerlessFinance-Api.ApiUrl = https://gl8fnu1vzi.execute-api.ap-southeast-1.amazonaws.com/prod/
ServerlessFinance-Api.ApiKeyId = 0kpl7aicf4
```

### Kiểm chứng - gọi thử API đã deploy

```bash
$ curl -i -X POST "https://gl8fnu1vzi.execute-api.ap-southeast-1.amazonaws.com/prod/backtests" \
    -H "Content-Type: application/json" -d '{}'
```

```
HTTP/1.1 403 Forbidden
{"message":"Forbidden"}
```

Không có header `x-api-key` → `403`, kiểm chứng trên API đã deploy thật. Với key hợp lệ (dùng `aws apigateway test-invoke-method`, chạy đúng integration mà không cần in secret value của key vào báo cáo này):

```bash
$ aws apigateway test-invoke-method --rest-api-id gl8fnu1vzi --resource-id <backtestsResourceId> \
    --http-method POST --body '{"symbol":"BTCUSDT","interval":"1h","strategy":"sma_crossover",
    "start_date":"2025-01-01","end_date":"2025-03-31",
    "param_grid":{"fast_period":[5,10],"slow_period":[30,50]}}'
```

```json
{"status": 202, "body": "{\"jobId\": \"ef275ad3-...\", \"executionArn\": \"arn:aws:states:...\", \"paramCount\": 4}"}
```

```bash
$ aws apigateway test-invoke-method --rest-api-id gl8fnu1vzi --resource-id <jobIdResourceId> \
    --http-method GET --path-with-query-string "/backtests/7fc7eb24-5bf3-41cd-9933-cbb2104bc55a"
```

```json
{"status": 200, "body": "{\"status\": \"SUCCEEDED\", \"param_count\": 4, \"result_count\": 4, \"symbol\": \"BTCUSDT\", ...}"}
```

```bash
$ aws apigateway test-invoke-method --rest-api-id gl8fnu1vzi --resource-id <resultsResourceId> \
    --http-method GET --path-with-query-string "/backtests/7fc7eb24-.../results?limit=2"
```

```json
{"status": 200, "body": "{\"results\": [{\"params\": {\"fast_period\": 5, \"slow_period\": 30}, \"sharpe_ratio\": -0.664, \"rank\": 0}, ...]}"}
```

```bash
$ aws apigateway test-invoke-method --rest-api-id gl8fnu1vzi --resource-id <jobIdResourceId> \
    --http-method GET --path-with-query-string "/backtests/does-not-exist"
```

```json
{"status": 404, "body": "{\"message\": \"Job 'does-not-exist' not found\"}"}
```

`403` (không có key), `202`/`200` (tạo/đọc), và `404` (job không tồn tại) đều kiểm chứng trên stage API Gateway đã deploy.

Tiếp tục tới [Phase 5 - Paper Trading Thời gian thực](../5.7-PaperTrading/).
