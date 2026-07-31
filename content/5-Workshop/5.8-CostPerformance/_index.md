---
title: "Phase 6 - Cost & Performance Optimization"
date: 2026-07-30
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
includeInReport: true
---

### Objective

The Worker Lambda from Phase 3 runs once per parameter combination. Three optimizations target that specific hot path: reading only the bytes actually needed from S3, caching across warm-container invocations, and replacing the equity-curve/metrics computation with a compiled Rust core.

### 1. S3 byte-range fetch + column projection - `services/common/data_access.py`

Switched from `s3.get_object` (downloads the whole Parquet object) to `pyarrow.fs.S3FileSystem` with column projection, the worker only needs `open_time`/`close`, so pyarrow issues HTTP Range GETs for just those column chunks:

```python
_S3FS = pafs.S3FileSystem(region=os.environ.get("AWS_REGION", "ap-southeast-1"))

def _read_partition(bucket, key, columns):
    cache_path = _cache_path(bucket, key, columns)
    if os.path.exists(cache_path):
        return pq.read_table(cache_path).to_pandas()          # 2. /tmp cache hit

    table = pq.read_table(f"{bucket}/{key}", columns=columns, filesystem=_S3FS)
    pq.write_table(table, cache_path)
    return table.to_pandas()
```

### 2. `/tmp` caching

Lambda execution environments are reused across invocations. A Distributed Map runs far more items than its concurrency limit, so the same warm container handles many worker invocations back to back. Param combinations that share a symbol/date range (the common case) skip S3 entirely after the first read.

### 3. Rust/PyO3 core - `rust/backtest_core/src/lib.rs`

A faithful, fast port of `engine.py`'s equity-curve/Sharpe/drawdown/win-rate math, built via `maturin` inside the Dockerfile's multi-stage build (no local Rust toolchain needed):

```dockerfile
FROM public.ecr.aws/lambda/python:3.11 AS rust-builder
RUN yum install -y gcc gcc-c++ && yum clean all
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --default-toolchain stable
ENV PATH="/root/.cargo/bin:${PATH}"
RUN pip install --no-cache-dir maturin
COPY rust/backtest_core /rust/backtest_core
WORKDIR /rust/backtest_core
RUN maturin build --release --interpreter python3.11 --out /wheels

FROM public.ecr.aws/lambda/python:3.11
COPY --from=rust-builder /wheels /wheels
RUN pip install --no-cache-dir /wheels/*.whl --target "${LAMBDA_TASK_ROOT}"
...
```

```rust
#[pyfunction]
fn run_backtest_metrics(close: Vec<f64>, signal: Vec<f64>, fee_rate: f64,
                         initial_capital: f64, periods_per_year: Option<f64>)
    -> PyResult<(f64, f64, f64, f64, usize)> {
    // executed_position = signal.shift(1); bar_returns = close.pct_change(); ...
    // equity curve, Sharpe, max drawdown, trade extraction — one tight Rust loop
}
```

`engine.py` itself is untouched and stays the canonical implementation for local/Phase 2 tests; `services/common/engine_rust.py` is a thin wrapper the worker calls instead.

### Deploy

```bash
cd infra
cdk deploy ServerlessFinance-Compute --require-approval never
```

```
 ServerlessFinance-Compute (Docker image rebuilt: worker now bundles the Rust wheel)
```

### Benchmark - Before vs. after

16-combination SMA grid, BTCUSDT 1h, full year of 2025 (≈8,760 bars), same 1024 MB memory. 16 sequential `aws lambda invoke` calls against the Worker Lambda directly (bypassing Step Functions, to isolate worker execution time), then a CloudWatch Logs Insights query over the REPORT lines:

```
fields @message | filter @message like /REPORT/
| parse @message "Billed Duration: * ms" as billed
| stats count(*) as n, avg(billed) as avgMs, max(billed) as maxMs, min(billed) as minMs
```

| | Before (pure Python) | After (byte-range + cache + Rust) |
|---|---|---|
| n | 16 | 16 |
| Avg Billed Duration | 419.4375 ms | 44.375 ms |
| Min / Max | 391 / 467 ms | 34 / 58 ms |

**→ ~9.4× faster, ~89% less Lambda cost** for the single most-invoked function in the system. Results were checked for numeric parity against the pre-change output for the same parameters: `total_return`/`max_drawdown`/`win_rate`/`num_trades` matched exactly; `sharpe_ratio` differed at the ~13th significant digit, ordinary floating-point summation-order noise, not a correctness issue.

{{% notice warning %}}
While benchmarking, we also discovered this AWS account's default Lambda concurrent-execution quota is only **10**, below the Distributed Map's configured `max_concurrency` of 50 (Phase 3). A large enough job can hit `Lambda.TooManyRequestsException`. Documented the fix is a quota increase request or a one-line `max_concurrency` change, applied only if actually needed.
{{% /notice %}}

Continue to [Phase 7 - Observability & Hardening](../5.9-Observability/).
