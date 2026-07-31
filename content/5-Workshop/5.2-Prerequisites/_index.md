---
title: "Prerequisites"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
includeInReport: true
---

### Accounts & tools

- An **AWS account** with an IAM user/role that has permission to create the resources used in this workshop (S3, Lambda, Step Functions, DynamoDB, API Gateway, Glue/Athena, EventBridge, CloudWatch, SNS, SQS, IAM roles). No AWS Free Tier–only restriction applies, but every service used here is either request-billed or on-demand.
- **AWS CLI**, configured with working credentials:

```bash
aws sts get-caller-identity
```

- **Python 3.11+**: the Lambda runtime version used throughout, and the version the local backtest engine/tests run under.
- **Node.js** (for the AWS CDK CLI):

```bash
npm install -g aws-cdk
```

- **Docker**: two of the eight Lambda functions (ingestion, worker) ship as container images because their dependencies (pandas + pyarrow + numpy) exceed Lambda's 250 MB zip code-size limit; the worker's Rust extension is also compiled inside a Docker multi-stage build, so no local Rust toolchain is required.
- **Git**.

### Project setup

```bash
git clone https://github.com/BlueWhaleo5/Serverless-Financial-Backtesting-Paper-Trading-Engine.git
cd Serverless-Financial-Backtesting-Paper-Trading-Engine

python -m venv .venv
.venv\Scripts\activate        # Windows
pip install -r requirements.txt
pip install -r infra/requirements.txt
```

### CDK bootstrap

CDK needs a one-time bootstrap per account/region to provision the S3 bucket and IAM roles the CDK CLI uses to stage deployment assets:

```bash
cd infra
cdk bootstrap
```

### Verify the toolchain

```bash
cdk synth        # renders every stack's CloudFormation template locally, no AWS calls that create resources
```

A clean `cdk synth` run (no errors, template files written to `infra/cdk.out/`) means the environment is ready. Continue to [Phase 1 - Data Lake](../5.3-DataLake/).
