---
title: "Phase 7 - Observability & Hardening"
date: 2026-07-30
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
includeInReport: true
---

### Mục tiêu

Thêm dashboard và alarm CloudWatch phủ toàn bộ Lambda và state machine, cùng một dead-letter queue cho Lambda duy nhất được gọi bất đồng bộ.

### Lambda nào thực sự cần DLQ?

Trong 8 Lambda function của hệ thống, đúng một cái được gọi bất đồng bộ: Ingestion Lambda, kích hoạt bởi lịch EventBridge hàng ngày. Mọi Lambda khác đều đứng sau một tích hợp proxy đồng bộ của API Gateway hoặc một Step Functions Task, lỗi ở đó sẽ trả thẳng về caller hoặc vào cơ chế retry/catch của Step Functions, nên DLQ sẽ dư thừa.

```python
# infra/stacks/data_lake_stack.py
self.ingestion_dlq = sqs.Queue(self, "IngestionDLQ", retention_period=Duration.days(14))

self.ingestion_fn = _lambda.DockerImageFunction(
    self, "IngestionFunction",
    ...
    dead_letter_queue_enabled=True,
    dead_letter_queue=self.ingestion_dlq,
    retry_attempts=2,
)
```

### Dashboard + alarm - `infra/stacks/observability_stack.py`

Stack này chỉ đọc metric theo tên/ARN function từ 4 stack còn lại, không cần IAM grant theo chiều nào:

```python
for name, fn in functions.items():
    errors = fn.metric_errors(period=Duration.minutes(5))
    if name in _ALARM_WORTHY:      # 6 Lambda thuộc "money path"
        cw.Alarm(self, f"{name}ErrorsAlarm", metric=errors, threshold=1,
                  evaluation_periods=1, treat_missing_data=cw.TreatMissingData.NOT_BREACHING,
        ).add_alarm_action(cw_actions.SnsAction(self.alarm_topic))

# Throttle của Worker đặc biệt quan trọng: quota concurrency của account (10, xem
# Phase 6) thấp hơn max_concurrency của Distributed Map (50).
worker_throttles = functions["worker_fn"].metric_throttles(period=Duration.minutes(5))
cw.Alarm(self, "WorkerThrottlesAlarm", metric=worker_throttles, threshold=1, ...)
```

### Deploy

```bash
cd infra
cdk deploy ServerlessFinance-Observability --require-approval never
```

```
 ServerlessFinance-Observability
Outputs:
ServerlessFinance-Observability.AlarmTopicArn = arn:aws:sns:ap-southeast-1:752643409325:ServerlessFinance-Alarms
ServerlessFinance-Observability.DashboardUrl = https://ap-southeast-1.console.aws.amazon.com/cloudwatch/home?region=ap-southeast-1#dashboards:name=ServerlessFinance
```

### Kiểm chứng

```bash
$ aws cloudwatch describe-alarms --alarm-name-prefix "ServerlessFinance-Observability" \
  --query "MetricAlarms[].{Name:AlarmName,State:StateValue}" --output table
```

```
---------------------------------------------------------------------------------------
|                                  DescribeAlarms                                     |
+----------------------------------------------------------------------------+--------+
|                                  Name                                      | State  |
+----------------------------------------------------------------------------+--------+
|ServerlessFinance-Observability-StateMachineFailedAlarm79113CC8-fpPihsgIPbxu  |  OK  |
|ServerlessFinance-Observability-WorkerThrottlesAlarm3D9E14E4-agaA3IUYpkOD     |  OK  |
|ServerlessFinance-Observability-aggregatorfnErrorsAlarm04C4440A-FzL6i6g9PFM2  |  OK  |
|ServerlessFinance-Observability-apifnErrorsAlarmAA581B67-XQ9jIsrwgvHI         |  OK  |
|ServerlessFinance-Observability-ingestionfnErrorsAlarm37C056BE-YjCcPIYUQPS1   |  OK  |
|ServerlessFinance-Observability-orchestratorfnErrorsAlarm02927765-PcwlDkjpFqWo|  OK  |
|ServerlessFinance-Observability-tradingfnErrorsAlarm979EF716-ighwKh7Nh7JO     |  OK  |
|ServerlessFinance-Observability-workerfnErrorsAlarm743BF9BE-ELqboWOZvxAO      |  OK  |
+----------------------------------------------------------------------------+--------+
```

Cả 8 alarm, mỗi cái cho một Lambda "money path", alarm throttle của Worker, và alarm lỗi state machine, báo `OK` ngay sau khi deploy, `treat_missing_data=NOT_BREACHING` coi đúng "chưa có lỗi nào" là khỏe mạnh thay vì thiếu dữ liệu. Dashboard (link `ServerlessFinance-Observability.DashboardUrl` ở trên) hiển thị đúng lỗi/duration của 8 Lambda cộng executions/execution-time của Step Functions, dựng từ chính các widget định nghĩa trong `observability_stack.py`.

Tiếp tục tới [Dọn dẹp tài nguyên](../5.10-Cleanup/).
