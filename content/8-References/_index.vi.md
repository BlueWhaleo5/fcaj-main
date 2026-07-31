---
title: "Tài liệu tham khảo"
date: 2026-07-30
weight: 8
chapter: false
pre: " <b> 8. </b> "
includeInReport: true
---

### Source Code

* **Repo GitHub** (source code đầy đủ: hạ tầng CDK, Lambda handler, lõi Rust, test): <https://github.com/BlueWhaleo5/Serverless-Financial-Backtesting-Paper-Trading-Engine>
* Toàn bộ code snippet và đoạn trích CDK/Dockerfile trong phần [Bản đề xuất](../2-Proposal/) và [Workshop](../5-Workshop/) đều lấy trực tiếp từ repo.

### Video demo

* *[[DEMO] Serverless Financial Backtesting & Paper Trading Engine](https://youtu.be/BNKZH79PfxY)*

### Tài liệu tham khảo đã dùng

Tài liệu AWS đã tham khảo trong quá trình xây dựng từng phase:

* AWS CDK (Python) - <https://docs.aws.amazon.com/cdk/api/v2/python/>
* AWS Lambda (container image, dead-letter queue) - <https://docs.aws.amazon.com/lambda/>
* AWS Step Functions, Distributed Map - <https://docs.aws.amazon.com/step-functions/>
* Amazon API Gateway (REST API, WebSocket API) - <https://docs.aws.amazon.com/apigateway/>
* Amazon DynamoDB - <https://docs.aws.amazon.com/dynamodb/>
* AWS Glue / Amazon Athena (partition projection) - <https://docs.aws.amazon.com/athena/>
* Amazon CloudWatch (dashboard, alarm) - <https://docs.aws.amazon.com/cloudwatch/>
* AWS Study Group - Cloud Journey (Tuần 1) - <https://cloudjourney.awsstudygroup.com/>

### API & công cụ bên ngoài

* **Binance Public API** - dữ liệu klines lịch sử (`/api/v3/klines`) và giá ticker thời gian thực (`/api/v3/ticker/price`), dùng cho dữ liệu OHLCV và khớp lệnh paper trading. <https://binance-docs.github.io/apidocs/spot/en/>
* **PyO3 / maturin** - binding Rust↔Python và công cụ build cho phần lõi hiệu năng ở Phase 6. <https://pyo3.rs/>, <https://www.maturin.rs/>
* **pandas / pyarrow / numpy** - xử lý dữ liệu trong Ingestion và Worker Lambda.

### Chương trình FCAJ

* First Cloud AI Journey - cộng đồng AWS Study Group: <https://awsstudygroup.com>, nhóm Facebook: <https://www.facebook.com/groups/awsstudygroupfcj>
