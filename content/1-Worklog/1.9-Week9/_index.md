---
title: "Week 9 Worklog"
date: 2026-07-30
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
includeInReport: true
reportTableColumns:
  - Day
  - Task
  - Completion Date
reportHeadings:
  - Week 9 Objectives
  - Tasks to be carried out this week
  - Week 9 Achievements
reportType: worklog
---

### Week 9 Objectives:

* Start Phase 6 - cost & performance optimization of the Worker Lambda hot path.
* Capture a "before" performance baseline, then implement S3 byte-range fetch and `/tmp` caching.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | Benchmark the current (pure-Python) worker: 16-combination job, capture CloudWatch Billed Duration | 29/06/2026 | 29/06/2026 | |
| 2 | Learn why `s3.get_object` downloads the whole Parquet file; study `pyarrow.fs.S3FileSystem` column projection | 30/06/2026 | 01/07/2026 | pyarrow docs |
| 3 | Rewrite `data_access.py` to fetch only the needed columns via true HTTP Range GETs | 01/07/2026 | 01/07/2026 | |
| 4 | Add `/tmp` caching keyed by bucket + key + columns, exploiting Lambda warm-container reuse | 02/07/2026 | 02/07/2026 | |
| 5 | Update the Worker Lambda to request only `["open_time", "close"]` | 03/07/2026 | 03/07/2026 | |

### Week 9 Achievements:

* Real "before" baseline captured on AWS: 419.4 ms average Billed Duration across 16 sequential worker invocations.
* S3 byte-range fetch + `/tmp` caching implemented.
* Ready to add the Rust core next week and re-measure.
