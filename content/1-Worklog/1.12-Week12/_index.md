---
title: "Week 12 Worklog"
date: 2026-07-30
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
includeInReport: true
reportTableColumns:
  - Day
  - Task
  - Completion Date
reportHeadings:
  - Week 12 Objectives
  - Tasks to be carried out this week
  - Week 12 Achievements
reportType: worklog
---

### Week 12 Objectives:

* Do a final full-system verification pass across all 8 phases on real AWS, then tear everything down to stop incurring cost.
* Write up the project as this FCAJ internship report (Proposal + Workshop + Worklog).

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | `cdk deploy --all`; troubleshoot a Docker Desktop engine hiccup mid-deploy, redeploy cleanly | 20/07/2025 | 20/07/2025 | |
| 2 | Re-run every phase's verification end-to-end: ingest data, run a Step Functions job, call the REST API, paper-trade with a live WebSocket push, check all 8 alarms | 21/07/2025 | 21/07/2025 | |
| 3 | `cdk destroy --all`, then manually empty/delete the `RemovalPolicy.RETAIN` S3 bucket and DynamoDB tables, confirm the account is back to zero ServerlessFinance resources | 22/07/2025 | 22/07/2025 | |
| 4 | Write the Proposal section | 23/07/2025 | 23/07/2025 | |
| 5 | Write the Workshop section | 24/07/2025 | 24/07/2025 | |
| 6 | Write this Worklog, fill in the Self-evaluation and Feedback sections| 25/07/2025 | 25/07/2025 | |

### Week 12 Achievements:

* Confirmed that every phase still works end-to-end after 12 weeks of incremental changes.
* Completed FCAJ internship report.