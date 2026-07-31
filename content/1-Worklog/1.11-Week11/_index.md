---
title: "Week 11 Worklog"
date: 2026-07-30
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
includeInReport: true
reportTableColumns:
  - Day
  - Task
  - Completion Date
reportHeadings:
  - Week 11 Objectives
  - Tasks to be carried out this week
  - Week 11 Achievements
reportType: worklog
---

### Week 11 Objectives:

* Build Phase 7 - a CloudWatch dashboard and alarm set covering every Lambda and the state machine.
* Add a dead-letter queue to the one Lambda in the system invoked asynchronously.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | Audit every Lambda's invocation model (sync API Gateway proxy / Step Functions Task vs. async EventBridge) to find which one actually needs a DLQ | 13/07/2025 | 13/07/2025 | |
| 2 | Add an SQS DLQ + `retry_attempts` to the ingestion Lambda | 13/07/2025 | 14/07/2025 | |
| 3 | Design `observability_stack.py`: reads metrics by function name/ARN from the other four stacks, no IAM coupling either direction | 14/07/2025 | 15/07/2025 | |
| 4 | Build the CloudWatch dashboard (errors/duration per Lambda, Step Functions executions) and 8 alarms, wired to an SNS topic | 15/07/2025 | 17/07/2025 | AWS CloudWatch docs |
| 5 | Deploy, verify all 8 alarms report `OK`, and confirm/troubleshoot the SNS email subscription | 17/07/2025 | 18/07/2025 | |

### Week 11 Achievements:

* Phase 7 deployed and verified: all 8 alarms `OK`, dashboard rendering real metrics, SQS DLQ in place on the ingestion Lambda.
* **All 8 phases of ServerlessFinance now complete** 
* Full write-up: [Workshop - Phase 7](../../5-Workshop/5.9-Observability/).
