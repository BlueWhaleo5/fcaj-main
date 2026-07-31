---
title: "Week 7 Worklog"
date: 2026-07-30
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
includeInReport: true
reportTableColumns:
  - Day
  - Task
  - Completion Date
reportHeadings:
  - Week 7 Objectives
  - Tasks to be carried out this week
  - Week 7 Achievements
reportType: worklog
---

### Week 7 Objectives:

* Deploy Phase 4 (API layer) and verify it with `curl` calls.
* Start Phase 5 - the real-time paper-trading module's design and data layer.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | `cdk deploy ServerlessFinance-Api`; verify create-job/status/results/403/404 with `curl` | 15/06/2025 | 15/06/2025 | |
| 2 | Design the paper-trading data model: accounts, positions, orders, cash balance | 16/06/2025 | 16/06/2025 | |
| 3 | Evaluate ElastiCache vs. DynamoDB for this state. Decide on DynamoDB on-demand for near-zero idle cost | 17/06/2025 | 17/06/2025 | |
| 4 | Implement order matching: MARKET fills immediately at live price, LIMIT fills only if already crossed | 18/06/2025 | 18/06/2025 | Binance API docs (`/ticker/price`) |
| 5 | Implement cash/position update logic with a 0.1% fee, matching Phase 2's engine assumption | 19/06/2025 | 19/06/2025 | |

### Week 7 Achievements:

* Phase 4 deployed and verified: real API calls returning `201`/`200`/`403`/`404` as expected.
* Key architecture decision made and documented: ElastiCache → DynamoDB on-demand for paper-trading state (cost-driven) (see [Proposal §4](../../2-Proposal/#4-technical-implementation)).
* `services/trading/handler.py`'s order-matching and fill logic implemented, not yet deployed.
