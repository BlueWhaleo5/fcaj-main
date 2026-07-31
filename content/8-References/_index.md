---
title: "References"
date: 2026-07-30
weight: 8
chapter: false
pre: " <b> 8. </b> "
includeInReport: true
---

### Source Code

* **GitHub repository** (full source: CDK infrastructure, Lambda handlers, Rust core, tests): <https://github.com/BlueWhaleo5/Serverless-Financial-Backtesting-Paper-Trading-Engine>
* All code snippets and CDK/Dockerfile excerpts shown in the [Proposal](../2-Proposal/) and [Workshop](../5-Workshop/) sections are taken directly from the repository.

### Demo Video

* *[[DEMO] Serverless Financial Backtesting & Paper Trading Engine](https://youtu.be/BNKZH79PfxY)*

### Documentation Referenced

AWS service documentation consulted while building each phase:

* AWS CDK (Python) - <https://docs.aws.amazon.com/cdk/api/v2/python/>
* AWS Lambda (container images, dead-letter queues) - <https://docs.aws.amazon.com/lambda/>
* AWS Step Functions, Distributed Map - <https://docs.aws.amazon.com/step-functions/>
* Amazon API Gateway (REST API, WebSocket API) - <https://docs.aws.amazon.com/apigateway/>
* Amazon DynamoDB - <https://docs.aws.amazon.com/dynamodb/>
* AWS Glue / Amazon Athena (partition projection) - <https://docs.aws.amazon.com/athena/>
* Amazon CloudWatch (dashboards, alarms) - <https://docs.aws.amazon.com/cloudwatch/>
* AWS Study Group - Cloud Journey (Week 1) - <https://cloudjourney.awsstudygroup.com/>

### External APIs & Tools

* **Binance Public API** - historical klines (`/api/v3/klines`) and live ticker price (`/api/v3/ticker/price`), used for OHLCV data and paper-trading fills. <https://binance-docs.github.io/apidocs/spot/en/>
* **PyO3 / maturin** - Rust↔Python bindings and build tooling for the Phase 6 performance core. <https://pyo3.rs/>, <https://www.maturin.rs/>
* **pandas / pyarrow / numpy** - data handling in the ingestion and worker Lambdas.

### FCAJ Program

* First Cloud AI Journey - AWS Study Group community: <https://awsstudygroup.com>, Facebook group: <https://www.facebook.com/groups/awsstudygroupfcj>
