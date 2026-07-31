---
title: "Proposal"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 2. </b> "
includeInReport: true
---

# ServerlessFinance - A Serverless Financial Backtesting & Paper Trading Engine

## A fully serverless AWS platform for parallel strategy backtesting and real-time paper trading

### 1. Executive Summary

ServerlessFinance is a serverless platform, built entirely on AWS, that lets a quant/crypto trader backtest a trading strategy across a large grid of parameters in parallel, and then paper-trade that strategy against live market prices without risking real capital. Historical OHLCV (Open/High/Low/Close/Volume) candle data is ingested from Binance's public API into an S3 data lake, a parameter grid (e.g. dozens of moving-average period combinations) is fanned out across many concurrent AWS Lambda workers via a Step Functions Distributed Map, and results are ranked by Sharpe ratio and served through a REST API. A second, independent module simulates paper-trading orders against live prices and pushes fills to connected clients in real time over a WebSocket. The whole system was designed, built, deployed, and verified end-to-end in eight incremental phases, each one shipped, tested against AWS infrastructure, and torn down/rebuilt as needed to control cost.

### 2. Problem Statement

**What's the Problem?**

Testing a trading strategy properly means running it across many parameter combinations (e.g. every fast/slow moving-average pair) over years of historical data, a workload that is naturally *embarrassingly parallel* but, run sequentially on a single machine, becomes slow as the parameter grid or the history grows. At the same time, a trader wants to validate a promising strategy against *live* prices before committing real money, which needs a persistent, always-available paper-trading account and real-time feedback, infrastructure that is overkill to run 24/7 on a personal machine or a rented VM just to backtest occasionally.

**The Solution**

ServerlessFinance replaces "one machine running backtests sequentially" with "hundreds of Lambda invocations running one parameter combination each, in parallel, for the duration of the job only". A Step Functions Distributed Map fans a job's parameter grid out across Worker Lambdas, an Aggregator ranks the results, and a REST API exposes job status and top results. A separate, equally serverless module lets the same trader place simulated market/limit orders that fill against live Binance prices, tracks cash/positions in DynamoDB, and pushes fill notifications over a WebSocket, at a much smaller and cheaper scale, the architecture a real trading platform would use.

### 3. Solution Architecture

The system is organized as five independent CDK stacks, each deployable and testable on its own, wired together only through the specific outputs each one actually needs (an S3 bucket reference, a Lambda function, a Step Functions state machine ARN):

![ServerlessFinance architecture](/images/2-Proposal/serverless_architecture.png)

- *Three dashed gray lines, labeled "25", run from the Worker Lambda, Step Functions (and other Lambdas) to CloudWatch. This is the metrics/logs stream. All Lambdas and Step Functions automatically send logs and metrics to CloudWatch in the background, not an explicit API call in the main stream. Because it runs in parallel and asynchronously with the main stream (not the "next step" in the sequence 1→27), the dashed lines are used to distinguish it.*

- *The vertical dashed gray line "On Failure" (from Ingestion Lambda down to SQS DLQ) is the error path, only used when the Ingestion Lambda fails (Lambda retrys but still fails), not a normal path.*

**AWS Services Used**
- **Amazon S3** - stores OHLCV data as partitioned Parquet (`symbol/interval/year/month`), the format the backtesting engine reads directly.
- **AWS Lambda** - every compute step: ingestion, orchestration, the backtest worker (with a Rust/PyO3-accelerated core, see Phase 6 below), aggregation, the two REST APIs, and the WebSocket connect/disconnect handlers.
- **AWS Step Functions (Distributed Map)** - fans a job's parameter grid out across parallel Worker Lambda invocations and aggregates the results, with built-in retry/catch so a failed job is marked `FAILED` instead of hanging.
- **Amazon DynamoDB** (on-demand) - job/result state, paper-trading accounts/positions/orders/P&L, and WebSocket connection tracking. Chosen over the originally-planned ElastiCache/Timestream specifically to stay near $0 when idle (see Section 4).
- **Amazon API Gateway** - a REST API (API-key gated) for the backtesting engine, a second REST API for paper trading, and a WebSocket API for real-time order-fill pushes.
- **AWS Glue + Amazon Athena** - a partition-projected catalog over the S3 data lake, so the ingested data can be spot-checked with plain SQL without a crawler.
- **Amazon EventBridge** - a daily schedule that keeps a watchlist of symbols incrementally up to date.
- **Amazon CloudWatch + Amazon SNS** - a dashboard and 8 alarms (Lambda errors, Worker throttles, Step Functions failures) across every Lambda and the state machine, wired to an SNS topic.
- **Amazon SQS** - a dead-letter queue on the one Lambda invoked asynchronously (the EventBridge-triggered ingestion function); every other Lambda is invoked synchronously and doesn't need one.

{{% notice note %}}
**No VPC, subnets, or Availability Zones anywhere in this stack.** Every service used is a fully-managed AWS service reached over the AWS API plane; none of them need to live inside a private network. A VPC becomes necessary once a workload has a resource that has to stay off the public internet - RDS, ElastiCache/Redis, or containers sitting behind an internal ALB, none of which this system uses. DynamoDB on-demand fills exactly the roles ElastiCache and Timestream would otherwise have played (see the deviations below).
{{% /notice %}}

**Component Design**
- **Data Lake**: the Ingestion Lambda fetches klines from Binance's REST API and writes them as Parquet, partitioned by symbol/interval/year/month, with a daily EventBridge schedule keeping a watchlist fresh.
- **Backtesting engine core**: a pure-Python module (`services/common/engine.py`) implementing the strategy interface, the bar-by-bar equity-curve simulator, and the summary metrics (Sharpe ratio, max drawdown, win rate), unit-tested locally with no AWS dependency.
- **Distributed compute**: an Orchestrator Lambda turns a request into a Step Functions execution; the Distributed Map runs one Worker Lambda per parameter combination; an Aggregator ranks results by Sharpe ratio into DynamoDB.
- **API layer**: a REST API in front of the Orchestrator (create job) and a read-only Lambda (status/results), gated by an API key.
- **Real-time paper trading**: a Trading Lambda matches simulated orders against Binance's live ticker price, updates DynamoDB, and pushes the fill to any WebSocket connections registered for that account.
- **Observability**: a CloudWatch dashboard and alarm set built as its own stack, reading metrics by name/ARN from the other four stacks with no IAM coupling in either direction.

### 4. Technical Implementation

**Implementation Phases**

The project was built and deployed to AWS infrastructure at the end of every phase:

| Phase | Scope |
|---|---|
| 0 | Project scaffolding: CDK app (Python), repo structure |
| 1 | Data Lake: Binance ingestion Lambda, S3 Parquet, Glue/Athena |
| 2 | Backtesting engine core: strategy interface, simulator, metrics (unit-tested) |
| 3 | Distributed compute: Step Functions Distributed Map, Orchestrator/Worker/Aggregator Lambdas |
| 4 | API layer: REST API + API-key auth |
| 5 | Real-time paper trading: order matching, DynamoDB state, WebSocket push |
| 6 | Cost & performance optimization: S3 byte-range fetch, `/tmp` caching, Rust/PyO3 core |
| 7 | Observability & hardening: CloudWatch dashboard/alarms, SQS DLQ |

**Two deliberate deviations from the original design**, both discovered and resolved while actually building on AWS rather than assumed up front:
- **ElastiCache → DynamoDB on-demand** for paper-trading state: ElastiCache Serverless bills for storage + ECPU even at zero traffic, while DynamoDB on-demand is effectively free when idle, the right tradeoff for a project that isn't run continuously.
- **Amazon Timestream → DynamoDB** for P&L history: Timestream for LiveAnalytics turned out to have **no CloudFormation/API presence in `ap-southeast-1`**, confirmed by an failed deployment (`Unrecognized resource types: [AWS::Timestream::Table, AWS::Timestream::Database]`), not assumed so P&L points are written to a DynamoDB table instead.

**Technical Requirements**
- AWS CDK (Python) for all infrastructure - one CDK app, five stacks, each independently deployable.
- Lambda runtimes: plain Python 3.11 for lightweight handlers (API, orchestrator, trading); container-image Lambdas for anything needing pandas/pyarrow (ingestion, worker), since those dependencies exceed the 250 MB zip code-size limit; a Rust/PyO3 extension (built via `maturin` inside a multi-stage Dockerfile) for the worker's hot-path metrics computation.
- Binance's public REST API (`/api/v3/klines` for history, `/api/v3/ticker/price` for live paper-trading fills) - free, no license/API-key requirements, well suited to a personal crypto-quant project.

### 5. Timeline & Milestones

- **Month 1**: AWS fundamentals, account/CLI setup, Phases 0–2 (scaffolding, data lake, backtest engine core).
- **Month 2**: Phases 3–5 (distributed compute, API layer, real-time paper trading) - the core end-to-end product.
- **Month 3**: Phases 6–7 (Rust/PyO3 performance optimization, observability/hardening), final verification, teardown, and documentation (this report).

### 6. Budget Estimation

Every AWS service used here is either serverless-request-priced (Lambda, API Gateway, Step Functions) or on-demand storage (S3, DynamoDB). The system was destroyed (`cdk destroy --all`) whenever not actively being built or demoed, so actual spend across the internship was a small fraction of even the light-usage estimate below.

**Estimated monthly cost at light personal-project usage** (a few dozen backtest jobs/month, occasional paper-trading activity, dashboard + 8 alarms always on):

| Service | Estimated monthly cost |
|---|---|
| Lambda (all functions, Rust-accelerated worker) | < $0.50 |
| API Gateway (2× REST + 1× WebSocket) | < $0.10 |
| Step Functions (Distributed Map) | < $0.10 |
| DynamoDB (on-demand, all 8 tables) | < $0.20 |
| S3 (OHLCV Parquet storage) | < $0.10 |
| CloudWatch (dashboard + 8 alarms) | ~$0.80 |
| SNS / SQS | negligible |
| **Total** | **≈ $1–2/month at active-use levels** |

This is consistent with what Phase 6's benchmarking actually measured: switching the Worker Lambda from pure Python to the byte-range-fetch + `/tmp`-cache + Rust-core combination cut its average billed duration from 419.4 ms to 44.4 ms per invocation, **9.4× speedup and ~89% reduction in Lambda compute cost** for the single most-invoked function in the system, measured on real AWS, not estimated.

### 7. Risk Assessment

**Risk Matrix**
- Third-party API availability (Binance): medium impact, low probability, the ingestion Lambda's daily schedule and manual-backfill path both degrade gracefully (a failed run doesn't corrupt already-ingested partitions).
- Account-level service quotas: medium impact, medium probability, discovered in practice during Phase 6 benchmarking, this AWS account's default Lambda concurrent-execution quota (10) is lower than the Distributed Map's configured `max_concurrency` (50), so a sufficiently large backtest job can throttle.
- Regional service availability: low impact, low probability once known, Amazon Timestream is not available in `ap-southeast-1`, discovered via a failed deploy in Phase 5.

**Mitigation Strategies**
- Binance dependency: an SQS dead-letter queue on the only asynchronously-invoked Lambda (ingestion) so a failed scheduled run isn't silently lost.
- Concurrency quota: documented in the project README as a known limitation; the fix (request a quota increase, or lower `max_concurrency`) is a one-line/one-ticket change, deliberately not applied speculatively before it's actually needed.
- Regional availability: verified empirically per AWS service rather than assumed from documentation, and designed around (DynamoDB) rather than worked around (a second region).

**Contingency Plans**
- Every stack can be destroyed and redeployed independently (`cdk destroy`/`cdk deploy` per stack), so a bad deploy is always recoverable within minutes.
- DynamoDB tables holding real backtest results or paper-trading state use `RemovalPolicy.RETAIN`, so a stack teardown never silently discards data that matters.

### 8. Expected Outcomes

**Technical Improvements**
A working, end-to-end serverless system verified on real AWS infrastructure at every phase, not just designed on paper: real backtest jobs executed through Step Functions, real paper-trading fills against live Binance prices, real CloudWatch alarms observed in the `OK` state, and a real, measured 9.4× performance improvement from the Rust optimization phase.

**Long-term Value**
The codebase and CDK stacks are reusable as a foundation for adding more strategies, more exchanges, or a real front-end; the documented deviations (DynamoDB instead of ElastiCache/Timestream, WebSocket API instead of AppSync) and the discovered account/region constraints are recorded so they don't have to be rediscovered on the next serverless project.
