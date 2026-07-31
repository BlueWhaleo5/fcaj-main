---
title: "Week 6 Worklog"
date: 2026-07-30
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
includeInReport: true
reportTableColumns:
  - Day
  - Task
  - Completion Date
reportHeadings:
  - Week 6 Objectives
  - Tasks to be carried out this week
  - Week 6 Achievements
reportType: worklog
---

### Week 6 Objectives:

* Deploy and verify Phase 3: run an actual multi-parameter backtest job through Step Functions.
* Start Phase 4 - a REST API in front of the Orchestrator, gated by an API key.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | `cdk deploy ServerlessFinance-Compute`; fix IAM permission issues found on first deploy | 08/06/2026 | 08/06/2026 | |
| 2 | Run backtest job, poll the Step Functions execution to `SUCCEEDED`, check ranked results in DynamoDB | 09/06/2026 | 09/06/2026 | |
| 3 | Learn API Gateway REST API basics: resources, methods, Lambda proxy integration | 10/06/2026 | 11/06/2026 | AWS API Gateway docs |
| 4 | Set up an API key + usage plan; wire `POST /backtests` to the Orchestrator | 12/06/2026 | 12/06/2026 | |
| 5 | Write the read-only API Lambda for `GET /backtests/{jobId}` and `GET /backtests/{jobId}/results` | 13/06/2026 | 13/06/2026 | |

### Week 6 Achievements:

* Phase 3 verified end-to-end: a job (success path and failure path both) ran through the Distributed Map.
* Phase 4's API Gateway REST API stack drafted, ready for deployment/verification.
* Full write-ups: [Workshop - Phase 3](../../5-Workshop/5.5-DistributedCompute/), [Workshop - Phase 4](../../5-Workshop/5.6-APILayer/).
