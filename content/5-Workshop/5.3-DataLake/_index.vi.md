---
title: "Phase 1 - Data Lake"
date: 2026-07-30
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
includeInReport: true
---

### Mục tiêu

Ingest dữ liệu nến OHLCV lịch sử từ REST API công khai của Binance và lưu thành Parquet có partition trên S3. Đồng thời dựng bảng Glue Catalog dùng partition projection để có thể kiểm tra dữ liệu bằng SQL thuần trong Athena, không cần chạy crawler.

### Kiến trúc

![Kiến trúc Phase 1 - Data Lake](/images/5-Workshop/phase1_data_lake.png)

Cấu trúc S3 (kiểu Hive, khớp với partition projection của bảng Glue):

```
ohlcv/symbol=<SYMBOL>/interval=<INTERVAL>/year=<YYYY>/month=<MM>/data.parquet
```

### Hạ tầng chính - `infra/stacks/data_lake_stack.py`

```python
self.data_bucket = s3.Bucket(
    self, "OhlcvDataBucket",
    versioned=True,
    lifecycle_rules=[s3.LifecycleRule(noncurrent_version_expiration=Duration.days(30))],
    block_public_access=s3.BlockPublicAccess.BLOCK_ALL,
    encryption=s3.BucketEncryption.S3_MANAGED,
    removal_policy=RemovalPolicy.RETAIN,
)

# pandas + pyarrow + numpy vượt giới hạn 250MB của zip package, nên
# ingestion function được đóng gói dạng container image.
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

Bảng Glue dùng **partition projection** (`projection.enabled: true`), nên Athena tự tính các partition hợp lệ từ pattern của S3 key thay vì cần crawler để khám phá chúng.

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

Không truyền `start_date`/`end_date` trong event nghĩa là "chế độ incremental" lấy 2 ngày gần nhất, đúng lời gọi mà lịch EventBridge hàng ngày thực hiện cho watchlist mặc định.

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

### Kiểm chứng - backfill một năm dữ liệu và query thử

```bash
$ aws lambda invoke \
  --function-name ServerlessFinance-DataLak-IngestionFunction3919F32-spmpkbHHWbbp \
  --payload '{"symbol":"BTCUSDT","interval":"1h","start_date":"2025-01-01","end_date":"2025-12-31"}' \
  --cli-binary-format raw-in-base64-out out.json

{"symbol": "BTCUSDT", "interval": "1h", "partitions_written": [
  "ohlcv/symbol=BTCUSDT/interval=1h/year=2025/month=01/data.parquet",
  "ohlcv/symbol=BTCUSDT/interval=1h/year=2025/month=02/data.parquet",
  ... (10 tháng còn lại) ...
  "ohlcv/symbol=BTCUSDT/interval=1h/year=2025/month=12/data.parquet"
]}
```

```bash
$ aws s3api list-objects-v2 --bucket serverlessfinance-datalake-... \
    --prefix "ohlcv/symbol=BTCUSDT/interval=1h/" --query "Contents[].{Key:Key,Size:Size}"
```

| Key | Size (byte) |
|---|---|
| .../year=2025/month=01/data.parquet | 70451 |
| .../year=2025/month=02/data.parquet | 64517 |
| .../year=2025/month=06/data.parquet | 68184 |
| .../year=2025/month=12/data.parquet | 68383 |
| ... (tổng 12 partition, mỗi tháng một file) | |

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

720 = 30 ngày × 24 giờ, đúng chính xác cho tháng 6/2025 ở độ phân giải 1h, xác nhận partition projection resolve đúng mà không cần crawler.

Tiếp tục tới [Phase 2 - Lõi Engine Backtest](../5.4-BacktestEngine/).
