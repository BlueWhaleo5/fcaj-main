---
title: "Workshop"
date: 2026-07-30
weight: 5
chapter: false
pre: " <b> 5. </b> "
includeInReport: false
---

# Build ServerlessFinance: A Serverless Backtesting & Paper Trading Engine on AWS

#### Overview

This workshop walks through building **ServerlessFinance** end-to-end on AWS: a data lake for historical crypto price data, a distributed backtesting engine that fans a strategy's parameter grid across parallel Lambda workers, a REST API in front of it, a real-time paper-trading module with a WebSocket push channel, a Rust-accelerated hot path, and a CloudWatch observability layer, each deployed and verified on a AWS before moving to the next.

#### Content

1. [Workshop Overview](5.1-Overview/)
2. [Prerequisites](5.2-Prerequisites/)
3. [Phase 1 - Data Lake](5.3-DataLake/)
4. [Phase 2 - Backtesting Engine Core](5.4-BacktestEngine/)
5. [Phase 3 - Distributed Compute](5.5-DistributedCompute/)
6. [Phase 4 - API Layer](5.6-APILayer/)
7. [Phase 5 - Real-time Paper Trading](5.7-PaperTrading/)
8. [Phase 6 - Cost & Performance Optimization](5.8-CostPerformance/)
9. [Phase 7 - Observability & Hardening](5.9-Observability/)
10. [Clean Up](5.10-Cleanup/)
