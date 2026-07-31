---
title: "Week 8 Worklog"
date: 2026-07-30
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
includeInReport: true
reportTableColumns:
  - Day
  - Task
  - Completion Date
reportHeadings:
  - Week 8 Objectives
  - Tasks to be carried out this week
  - Week 8 Achievements
reportType: worklog
---

### Week 8 Objectives:

* Add the WebSocket push channel for real-time order-fill notifications.
* Deploy Phase 5 and verify the full paper-trading flow, including a live WebSocket client.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | Design the WebSocket API: `$connect`/`$disconnect` routes, a connections table keyed by `connectionId` | 22/06/2026 | 23/06/2026 | AWS API Gateway WebSocket docs |
| 2 | Implement `trading_ws_connect`/`trading_ws_disconnect` Lambdas | 23/06/2026 | 24/06/2026 | |
| 3 | Wire the Trading Lambda to push fills via `apigatewaymanagementapi.post_to_connection` | 24/06/2026 | 25/06/2026 | |
| 4 | First deploy attempt: hit a `cdk deploy` failure → Amazon Timestream has no CloudFormation presence in `ap-southeast-1` | 25/06/2026 | 25/06/2026 | |
| 5 | Redesign P&L history storage around DynamoDB instead of Timestream → redeploy successfully | 26/06/2026 | 26/06/2026 | |
| 6 | End-to-end verification: create account, place order, watch a `wscat`-style client receive the fill in real time | 27/06/2026 | 27/06/2026 | |

### Week 8 Achievements:

* Phase 5 fully deployed and verified on AWS: order fills, cash/position math, and real-time WebSocket push all confirmed against a live account.
* Lesson learned: Amazon Timestream for LiveAnalytics isn't available in `ap-southeast-1` → the fix (DynamoDB) is now the template for P&L-style history in this project.
* Full write-up: [Workshop - Phase 5](../../5-Workshop/5.7-PaperTrading/).
