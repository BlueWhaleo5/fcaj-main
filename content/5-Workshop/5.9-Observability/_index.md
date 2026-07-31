---
title: "Phase 7 - Observability & Hardening"
date: 2026-07-30
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
includeInReport: true
---

### Objective

Add a CloudWatch dashboard and alarms covering every Lambda and the state machine, and a dead-letter queue on the one Lambda that's invoked asynchronously.

### Which Lambda actually needs a DLQ?

Out of eight Lambda functions in the system, exactly one is invoked asynchronously: the ingestion Lambda, triggered by the daily EventBridge schedule. Every other Lambda sits behind a synchronous API Gateway proxy integration or a Step Functions Task, a failure there surfaces directly to the caller or to Step Functions' own retry/catch, so a DLQ would be redundant.

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

### Dashboard + alarms - `infra/stacks/observability_stack.py`

This stack only reads metrics by function name/ARN from the other four stacks, no IAM grants needed in either direction:

```python
for name, fn in functions.items():
    errors = fn.metric_errors(period=Duration.minutes(5))
    if name in _ALARM_WORTHY:      # the 6 "money path" Lambdas
        cw.Alarm(self, f"{name}ErrorsAlarm", metric=errors, threshold=1,
                  evaluation_periods=1, treat_missing_data=cw.TreatMissingData.NOT_BREACHING,
        ).add_alarm_action(cw_actions.SnsAction(self.alarm_topic))

# Worker throttles specifically matter: the account's concurrency quota (10, see
# Phase 6) is below the Distributed Map's max_concurrency (50).
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

### Verify

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

All 8 alarms, one per "money path" Lambda, the Worker's throttle alarm, and the state machine failure alarm, report `OK` right after deploy, `treat_missing_data=NOT_BREACHING` correctly treating "no errors yet" as healthy rather than as missing data. The dashboard (`ServerlessFinance-Observability.DashboardUrl` above) renders the same eight Lambdas' errors/duration plus Step Functions executions/execution-time, built from the exact widget definitions in `observability_stack.py`.

Continue to [Clean Up](../5.10-Cleanup/).
