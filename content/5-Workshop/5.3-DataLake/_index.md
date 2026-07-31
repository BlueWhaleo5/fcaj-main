---
title: "Phase 1 - Data Lake"
date: 2026-07-30
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
includeInReport: true
---

### Objective

Ingest historical OHLCV (candle) data from Binance's public REST API and store it as partitioned Parquet in S3. Also stand up a Glue Catalog table with partition projection so the data can be spot-checked with plain SQL in Athena, without running a crawler.

### Architecture

![Phase 1 - Data Lake architecture](/images/5-Workshop/phase1_data_lake.png)

S3 layout (Hive-style, matches the Glue table's partition projection):

```
ohlcv/symbol=<SYMBOL>/interval=<INTERVAL>/year=<YYYY>/month=<MM>/data.parquet
```

### Key infrastructure - `infra/stacks/data_lake_stack.py`

```python
self.data_bucket = s3.Bucket(
    self, "OhlcvDataBucket",
    versioned=True,
    lifecycle_rules=[s3.LifecycleRule(noncurrent_version_expiration=Duration.days(30))],
    block_public_access=s3.BlockPublicAccess.BLOCK_ALL,
    encryption=s3.BucketEncryption.S3_MANAGED,
    removal_policy=RemovalPolicy.RETAIN,
)

# pandas + pyarrow + numpy unzip to well over Lambda's 250MB zip code-size
# limit, so the ingestion function ships as a container image instead.
self.ingestion_fn = _lambda.DockerImageFunction(
    self, "IngestionFunction",
    code=_lambda.DockerImageCode.from_image_asset(directory="../services/ingestion"),
    timeout=Duration.minutes(15),
    memory_size=1024,
    environment={"BUCKET_NAME": self.data_bucket.bucket_name},
)

schedule_rule = events.Rule(
    self, "DailyIngestionSchedule",
    schedule=events.Schedule.cron(minute="0", hour="1"),
)
for symbol, interval in WATCHLIST:
    schedule_rule.add_target(targets.LambdaFunction(
        self.ingestion_fn,
        event=events.RuleTargetInput.from_object({"symbol": symbol, "interval": interval}),
    ))
```

The Glue table uses **partition projection** (`projection.enabled: true`), so Athena computes valid partitions from the S3 key pattern instead of needing a crawler to discover them.

### Ingestion Lambda - `services/ingestion/handler.py`

```python
def handler(event, context):
    symbol = event["symbol"].upper()
    interval = event["interval"]

    now_ms = int(datetime.now(timezone.utc).timestamp() * 1000)
    start_ms = _parse_date(event["start_date"]) if event.get("start_date") else now_ms - 2 * 86_400_000
    end_ms = _parse_date(event["end_date"]) if event.get("end_date") else now_ms

    df = fetch_klines(symbol, interval, start_ms, end_ms)
    ...
    for period, group in df.groupby("_period"):
        key = write_partition(group, symbol, interval, period.year, period.month)
```

No `start_date`/`end_date` in the event means "incremental mode" fetch the last 2 days, the same call the daily EventBridge schedule makes for the default watchlist.

### Deploy

```bash
cd infra
cdk deploy ServerlessFinance-DataLake --require-approval never
```

```
 ServerlessFinance-DataLake

 Deployment time: 133.43s

Outputs:
ServerlessFinance-DataLake.DataBucketName = serverlessfinance-datalake-ohlcvdatabucket2ec5f865-rhcuvefczb6j
ServerlessFinance-DataLake.GlueDatabaseName = serverless_finance
ServerlessFinance-DataLake.GlueTableName = ohlcv
ServerlessFinance-DataLake.IngestionDLQUrl = https://sqs.ap-southeast-1.amazonaws.com/752643409325/ServerlessFinance-DataLake-IngestionDLQF0F102AE-Irzccm2PAUMP
ServerlessFinance-DataLake.IngestionFunctionName = ServerlessFinance-DataLak-IngestionFunction3919F32-spmpkbHHWbbp
```

### Verify - backfill a year of data and query it

```bash
$ aws lambda invoke \
  --function-name ServerlessFinance-DataLak-IngestionFunction3919F32-spmpkbHHWbbp \
  --payload '{"symbol":"BTCUSDT","interval":"1h","start_date":"2025-01-01","end_date":"2025-12-31"}' \
  --cli-binary-format raw-in-base64-out out.json

{"symbol": "BTCUSDT", "interval": "1h", "partitions_written": [
  "ohlcv/symbol=BTCUSDT/interval=1h/year=2025/month=01/data.parquet",
  "ohlcv/symbol=BTCUSDT/interval=1h/year=2025/month=02/data.parquet",
  ... (10 more months) ...
  "ohlcv/symbol=BTCUSDT/interval=1h/year=2025/month=12/data.parquet"
]}
```

```bash
$ aws s3api list-objects-v2 --bucket serverlessfinance-datalake-... \
    --prefix "ohlcv/symbol=BTCUSDT/interval=1h/" --query "Contents[].{Key:Key,Size:Size}"
```

| Key | Size (bytes) |
|---|---|
| .../year=2025/month=01/data.parquet | 70451 |
| .../year=2025/month=02/data.parquet | 64517 |
| .../year=2025/month=06/data.parquet | 68184 |
| .../year=2025/month=12/data.parquet | 68383 |
| ... (12 partitions total, one per month) | |

```sql
SELECT COUNT(*) FROM serverless_finance.ohlcv
WHERE symbol='BTCUSDT' AND interval='1h' AND year='2025' AND month='06';
```

```bash
$ aws athena start-query-execution --query-string "..." \
    --result-configuration "OutputLocation=s3://.../athena-results/"
$ aws athena get-query-results --query-execution-id <id>
```

```
row_count
720
```

720 = 30 days × 24 hours, exactly right for June 2025 at 1h resolution, confirming the partition projection resolves correctly with no crawler involved.

Continue to [Phase 2 - Backtesting Engine Core](../5.4-BacktestEngine/).
