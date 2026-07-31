---
title: "Yêu cầu chuẩn bị"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
includeInReport: true
---

### Tài khoản & công cụ

- Một **tài khoản AWS** với IAM user/role có quyền tạo các tài nguyên dùng trong workshop (S3, Lambda, Step Functions, DynamoDB, API Gateway, Glue/Athena, EventBridge, CloudWatch, SNS, SQS, IAM role). Không giới hạn bởi AWS Free Tier, nhưng mọi dịch vụ dùng ở đây đều tính phí theo request hoặc on-demand.
- **AWS CLI**, đã cấu hình credentials hoạt động:

```bash
aws sts get-caller-identity
```

- **Python 3.11+**: phiên bản runtime dùng cho toàn bộ Lambda, và cũng là phiên bản chạy engine backtest/test cục bộ.
- **Node.js** (cho AWS CDK CLI):

```bash
npm install -g aws-cdk
```

- **Docker**: hai trong tám Lambda function (ingestion, worker) đóng gói dạng container image vì dependency của chúng (pandas + pyarrow + numpy) vượt giới hạn 250MB của zip package; extension Rust của worker cũng được biên dịch bên trong Docker multi-stage build, nên không cần cài Rust toolchain cục bộ.
- **Git**.

### Thiết lập project

```bash
git clone https://github.com/BlueWhaleo5/Serverless-Financial-Backtesting-Paper-Trading-Engine.git
cd Serverless-Financial-Backtesting-Paper-Trading-Engine

python -m venv .venv
.venv\Scripts\activate        # Windows
pip install -r requirements.txt
pip install -r infra/requirements.txt
```

### CDK bootstrap

CDK cần bootstrap một lần cho mỗi account/region để tạo S3 bucket và IAM role mà CDK CLI dùng để lưu tạm asset khi deploy:

```bash
cd infra
cdk bootstrap
```

### Kiểm tra toolchain

```bash
cdk synth        # dựng CloudFormation template của từng stack cục bộ, không gọi AWS để tạo tài nguyên nào
```

`cdk synth` chạy sạch (không lỗi, template được ghi vào `infra/cdk.out/`) nghĩa là môi trường đã sẵn sàng. Tiếp tục tới [Phase 1 - Data Lake](../5.3-DataLake/).
