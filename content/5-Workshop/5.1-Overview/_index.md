---
title: "Workshop Overview"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
includeInReport: true
---

### What we're building

**ServerlessFinance** is a serverless AWS platform with two independent modules that share the same historical price data:

1. **Backtesting engine**: ingest historical OHLCV (candle) data for a crypto pair, then run a trading strategy across a large grid of parameters (e.g. every combination of fast/slow moving-average period) in parallel, ranking results by Sharpe ratio.
2. **Real-time paper trading**: place simulated market/limit orders that fill against live prices, track cash and positions, and push fill notifications to a connected client over a WebSocket.

Both run without a single server, container, or database instance running in the background, everything is either request-billed (Lambda, API Gateway, Step Functions) or on-demand storage (S3, DynamoDB).

![ServerlessFinance architecture](/images/5-Workshop/serverless_architecture.png)

### Repository layout

```
ServerlessFinance/
├── infra/                  # AWS CDK app (Python) — 5 independent stacks
│   ├── app.py
│   └── stacks/
│       ├── data_lake_stack.py       # Phase 1
│       ├── compute_stack.py         # Phase 3
│       ├── api_stack.py             # Phase 4
│       ├── realtime_stack.py        # Phase 5
│       └── observability_stack.py   # Phase 7
├── services/                # Lambda handlers
│   ├── ingestion/ orchestrator/ worker/ aggregator/ api/
│   ├── trading/ trading_ws_connect/ trading_ws_disconnect/
│   └── common/               # shared engine, data access, Rust wrapper
├── strategies/               # pluggable strategy implementations
├── rust/backtest_core/        # Rust/PyO3 hot-path core (Phase 6)
└── tests/                    # local unit tests, no AWS required
```

### How this workshop is organized

Each of the following pages corresponds to one phase of the build: it was implemented, deployed with `cdk deploy` to AWS account, and verified against AWS infrastructure before moving on to the next phase.

By the end, we will have:
- A working data lake ingesting real Binance market data.
- A distributed backtest engine that can rank hundreds of parameter combinations in parallel.
- A REST API and a real-time paper-trading module with a WebSocket push channel.
- A Rust-accelerated worker that is ~9.4× faster than the pure-Python version.
- A CloudWatch dashboard and alarms watching the whole system.

Continue to [Prerequisites](../5.2-Prerequisites/) to set up the environment.
