---
title: "Week 3 Worklog"
date: 2026-07-30
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
includeInReport: true
reportTableColumns:
  - Day
  - Task
  - Completion Date
reportHeadings:
  - Week 3 Objectives
  - Tasks to be carried out this week
  - Week 3 Achievements
reportType: worklog
---

### Week 3 Objectives:

* Build Phase 1 - the Data Lake: ingest OHLCV data from Binance and store it as partitioned Parquet in S3.
* Stand up an Athena/Glue catalog to query the ingested data without a crawler.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | Study Binance's public REST API (`/api/v3/klines`) and design the S3 partition layout | 18/05/2026 | 18/05/2026 | Binance API docs |
| 2 | Write the ingestion Lambda (fetch klines, convert to Parquet with pandas/pyarrow) | 18/05/2026 | 18/05/2026 | |
| 3 | Learn why pandas + pyarrow needs a container-image Lambda (250MB zip limit); write the Dockerfile | 19/05/2026 | 20/05/2026 | AWS Lambda docs |
| 4 | Define the `DataLakeStack` in CDK: S3 bucket, DockerImageFunction, EventBridge daily schedule | 20/05/2026 | 21/05/2026 | |
| 5 | Add Glue Catalog table with partition projection; deploy and backfill a year of BTCUSDT data | 22/05/2026 | 22/05/2026 | |
| 6 | Verify via Athena: `SELECT COUNT(*)` per partition matches expected row counts | 23/05/2026 | 23/05/2026 | |

### Week 3 Achievements

* Phase 1 deployed and verified on AWS: `cdk deploy ServerlessFinance-DataLake` succeeds, ingestion Lambda backfills 12 months of BTCUSDT 1h data.
* Athena query against a partition-projected table (no crawler) returns the correct row count (720 rows for a 30-day, 1h-interval month).
* Full write-up: [Workshop - Phase 1 Data Lake](../../5-Workshop/5.3-DataLake/).
