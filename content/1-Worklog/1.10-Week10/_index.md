---
title: "Week 10 Worklog"
date: 2026-07-30
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
includeInReport: true
reportTableColumns:
  - Day
  - Task
  - Completion Date
reportHeadings:
  - Week 10 Objectives
  - Tasks to be carried out this week
  - Week 10 Achievements
reportType: worklog
---

### Week 10 Objectives:

* Learn just enough Rust + PyO3 to port the backtest engine's metrics computation.
* Build it inside a Docker multi-stage image (no local Rust toolchain), deploy, and re-benchmark.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | Learn PyO3 basics: `#[pyfunction]`, `#[pymodule]`, building a wheel with `maturin` | 06/07/2026 | 06/07/2026 | PyO3 / maturin docs |
| 2 | Port `engine.py`'s equity-curve/Sharpe/drawdown/trade-extraction math to Rust, faithfully (including a latent Python negative-index quirk) | 07/07/2026 | 08/07/2026 | |
| 3 | Write the multi-stage `Dockerfile`: a `rust-builder` stage (yum install gcc, rustup, maturin build) feeding the final Lambda image | 08/07/2026 | 08/07/2026 | |
| 4 | Debug the base image's package manager (`dnf` doesn't exist on this Lambda base image, it's `yum`) | 09/07/2026 | 10/07/2026 | |
| 5 | Deploy, smoke-test one param set for numeric parity against the pre-change output, then re-run the 16-combination benchmark | 11/07/2026 | 11/07/2026 | |

### Week 10 Achievements:

* Rust/PyO3 core (`rust/backtest_core`) built and deployed inside the worker's Docker image, no cross-compilation needed from the dev machine.
* **9.4× speedup measured on real AWS**: 419.4 ms → 44.4 ms average Billed Duration, ~89% less Lambda compute cost.
* Numeric parity confirmed against the pre-change Python output for the same parameters.
* Full write-up with the exact Dockerfile and benchmark numbers: [Workshop - Phase 6](../../5-Workshop/5.8-CostPerformance/).
