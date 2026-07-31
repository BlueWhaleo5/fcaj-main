---
title: "Dọn dẹp tài nguyên"
date: 2026-07-30
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
includeInReport: true
---

Mọi dịch vụ trong stack đều tính phí theo request/on-demand, nhưng để hệ thống deploy vô thời hạn vẫn không hoàn toàn miễn phí. Vài bảng DynamoDB và alarm CloudWatch vẫn phát sinh chi phí nhỏ nhưng khác 0, và nên tháo dỡ hạ tầng không đang được dùng/demo tích cực.

### Destroy các stack

`cdk destroy --all` xóa 5 stack theo thứ tự dependency ngược:

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

### Cái gì cố tình được giữ lại

Hai loại tài nguyên **không** bị xóa bởi `cdk destroy`, theo thiết kế:

```python
removal_policy=RemovalPolicy.RETAIN   # data_bucket, jobs_table, results_table,
                                       # accounts_table, positions_table, orders_table
```

Việc này bảo vệ kết quả backtest và dữ liệu paper trading thật khỏi bị xóa mất bởi một lần tháo dỡ stack vô tình hoặc thường quy.

```bash
# Làm trống S3 bucket có versioning (phải xóa mọi object version + delete
# marker trước khi xóa được bucket)
aws s3api list-object-versions --bucket <bucket> --output json | \
  jq '{Objects: ([.Versions[]?, .DeleteMarkers[]?] | map({Key,VersionId})), Quiet: true}' \
  > delete_payload.json
aws s3api delete-objects --bucket <bucket> --delete file://delete_payload.json
aws s3api delete-bucket --bucket <bucket>

# Xóa các bảng DynamoDB có RemovalPolicy.RETAIN
aws dynamodb delete-table --table-name <table-name>
```

### Kiểm chứng

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

Mọi query đều trả về danh sách rỗng. Không còn CloudFormation stack, S3 bucket, bảng DynamoDB, hay Lambda function nào của dự án này sót lại trong tài khoản.

### Deploy lại sau này

Để dựng lại:

```bash
cd infra
cdk deploy --all
```

Sau đó ingest lại dữ liệu OHLCV (Phase 1) trước khi chạy backtest, vì data lake cũng đã bị xóa cùng mọi thứ khác.

---

Workshop kết thúc tại đây. Anh chị có thể tham khảo [Bản đề xuất](../../2-Proposal/) để biết lý do thiết kế ban đầu và phân tích chi phí.
