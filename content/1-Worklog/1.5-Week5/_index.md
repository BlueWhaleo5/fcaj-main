---
title: "Week 5 Worklog"
date: 2026-07-30
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
includeInReport: true
reportTableColumns:
  - Day
  - Task
  - Completion Date
reportHeadings:
  - Week 5 Objectives
  - Tasks to be carried out this week
  - Week 5 Achievements
reportType: worklog
---

### Week 5 Objectives:

* Start Phase 3 - turn the local backtest engine into a distributed job using Step Functions Distributed Map.
* Implement the Worker Lambda and the Orchestrator Lambda.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | Study Step Functions Distributed Map: `items_path`, `item_selector`, `max_concurrency`, `ResultWriter` | 01/06/2026 | 01/06/2026 | AWS Step Functions docs |
| 2 | Write the Worker Lambda: read OHLCV from S3, run `engine.py`, return summary metrics | 02/06/2026 | 02/06/2026 | |
| 3 | Write the Orchestrator Lambda: expand a param grid into a Distributed Map item list, start an execution | 03/06/2026 | 03/06/2026 | |
| 4 | Design DynamoDB schema for job/result tracking (`BacktestJobsTable`, `BacktestResultsTable`) | 04/06/2026 | 04/06/2026 | |
| 5 | Wire up the Distributed Map's `add_catch` so a failed job is marked `FAILED`, not left hanging | 05/06/2026 | 06/06/2026 | |

### Week 5 Achievements:

* Worker and Orchestrator Lambdas implemented; Distributed Map state machine defined in `compute_stack.py`.
* Understood why zip-based `PythonFunction` doesn't work for the worker (pandas/pyarrow size) (reused the container-image pattern from Phase 1).
* Ready to deploy and verify next week.
