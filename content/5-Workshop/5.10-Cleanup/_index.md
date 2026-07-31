---
title: "Clean Up"
date: 2026-07-30
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
includeInReport: true
---

Every service in this stack is request/on-demand billed, but leaving the system deployed indefinitely still isn't free. A handful of DynamoDB tables and CloudWatch alarms accrue a small but nonzero cost, and it's good practice to tear down infrastructure that isn't actively being used or demoed.

### Destroy the stacks

`cdk destroy --all` removes the five stacks in reverse dependency order:

```bash
cd infra
cdk destroy --all --force
```

```
 ServerlessFinance-Observability: destroyed
 ServerlessFinance-Api: destroyed
 ServerlessFinance-Realtime: destroyed
 ServerlessFinance-Compute: destroyed
 ServerlessFinance-DataLake: destroyed
```

### What survives on purpose

Two kinds of resources are **not** deleted by `cdk destroy`, by design:

```python
removal_policy=RemovalPolicy.RETAIN   # data_bucket, jobs_table, results_table,
                                       # accounts_table, positions_table, orders_table
```

This protects real backtest results and paper-trading data from being discarded by an accidental or routine stack teardown.

```bash
# Empty the versioned S3 bucket (must delete every object version + delete marker
# before the bucket itself can be deleted)
aws s3api list-object-versions --bucket <bucket> --output json | \
  jq '{Objects: ([.Versions[]?, .DeleteMarkers[]?] | map({Key,VersionId})), Quiet: true}' \
  > delete_payload.json
aws s3api delete-objects --bucket <bucket> --delete file://delete_payload.json
aws s3api delete-bucket --bucket <bucket>

# Delete the RETAIN-policy DynamoDB tables
aws dynamodb delete-table --table-name <table-name>
```

### Verify

```bash
aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE \
  --query "StackSummaries[?contains(StackName,'ServerlessFinance')].StackName"
aws s3api list-buckets --query "Buckets[?contains(Name,'serverlessfinance')].Name"
aws dynamodb list-tables --query "TableNames[?contains(@,'ServerlessFinance')]"
aws lambda list-functions --query "Functions[?contains(FunctionName,'ServerlessFinance')].FunctionName"
```

```
=== stacks ===

=== buckets ===

=== dynamodb ===

=== lambda ===

all checks done
```

Every query returns an empty list. No CloudFormation stack, S3 bucket, DynamoDB table, or Lambda function belonging to this project remains in the account.

### Redeploying later

To bring it back:

```bash
cd infra
cdk deploy --all
```

Then re-ingest OHLCV data (Phase 1) before running any backtests, since the data lake was destroyed along with everything else.

---

This concludes the workshop. You may refer to the [Proposal](../../2-Proposal/) for the original design rationale and cost analysis.
