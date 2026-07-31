---
title: "Week 4 Worklog"
date: 2026-07-30
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
includeInReport: true
reportTableColumns:
  - Day
  - Task
  - Completion Date
reportHeadings:
  - Week 4 Objectives
  - Tasks to be carried out this week
  - Week 4 Achievements
reportType: worklog
---

### Week 4 Objectives:

* Build Phase 2 - the backtesting engine core, entirely local, no AWS dependency.
* Get it unit-tested against hand-computed numbers before wiring it into any distributed compute.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | Design the `Strategy` interface and a first strategy (SMA crossover) | 25/05/2026 | 26/05/2026 | |
| 2 | Implement the bar-by-bar equity-curve simulator with the shift-by-one execution model (no lookahead bias) | 26/05/2026 | 26/05/2026 | |
| 3 | Implement summary metrics: Sharpe ratio, max drawdown, win rate, trade extraction | 27/05/2026 | 27/05/2026 | |
| 4 | Write unit tests against a fixed synthetic OHLCV series with hand-computed expected values | 28/05/2026 | 29/05/2026 | pytest docs |
| 5 | Get `pytest` green locally, no AWS credentials required | 29/05/2026 | 29/05/2026 | |

### Week 4 Achievements:

* Phase 2 complete: `services/common/engine.py` + `strategies/sma_crossover.py`, fully unit-tested.
* `pytest` passes locally in a few second, this baseline is what Phase 6's Rust core is later checked against for numeric parity.
* Full write-up: [Workshop - Phase 2 Backtesting Engine Core](../../5-Workshop/5.4-BacktestEngine/).
