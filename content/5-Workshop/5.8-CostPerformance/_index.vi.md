---
title: "Phase 6 - Tối ưu Chi phí & Hiệu năng"
date: 2026-07-30
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
includeInReport: true
---

### Mục tiêu

Worker Lambda từ Phase 3 chạy một lần cho mỗi tổ hợp tham số. Ba tối ưu nhắm đúng vào hot path đó: chỉ đọc đúng byte cần từ S3, cache giữa các lần gọi trong cùng warm-container, và thay phần tính equity curve/metrics bằng lõi Rust biên dịch sẵn.

### 1. S3 byte-range fetch + column projection - `services/common/data_access.py`

Chuyển từ `s3.get_object` (tải nguyên object Parquet) sang `pyarrow.fs.S3FileSystem` với column projection, worker chỉ cần `open_time`/`close`, nên pyarrow chỉ gửi HTTP Range GET cho đúng các column chunk đó:

```python
_S3FS = pafs.S3FileSystem(region=os.environ.get("AWS_REGION", "ap-southeast-1"))

def _read_partition(bucket, key, columns):
    cache_path = _cache_path(bucket, key, columns)
    if os.path.exists(cache_path):
        return pq.read_table(cache_path).to_pandas()          # 2. cache /tmp hit

    table = pq.read_table(f"{bucket}/{key}", columns=columns, filesystem=_S3FS)
    pq.write_table(table, cache_path)
    return table.to_pandas()
```

### 2. Cache `/tmp`

Lambda execution environment được tái sử dụng giữa các lượt gọi. Distributed Map chạy nhiều item hơn nhiều so với giới hạn concurrency, nên cùng một warm container xử lý nhiều lượt gọi worker liên tiếp. Các tổ hợp tham số cùng symbol/khoảng ngày (trường hợp phổ biến nhất) bỏ qua hẳn S3 sau lần đọc đầu tiên.

### 3. Lõi Rust/PyO3 - `rust/backtest_core/src/lib.rs`

Bản port trung thành, nhanh của phần toán equity-curve/Sharpe/drawdown/win-rate trong `engine.py`, build qua `maturin` bên trong Dockerfile multi-stage (không cần cài Rust toolchain cục bộ):

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
    // equity curve, Sharpe, max drawdown, trích xuất trade — một vòng lặp Rust duy nhất
}
```

`engine.py` giữ nguyên, vẫn là bản chuẩn cho test cục bộ/Phase 2; `services/common/engine_rust.py` là một wrapper mỏng mà worker gọi thay thế.

### Deploy

```bash
cd infra
cdk deploy ServerlessFinance-Compute --require-approval never
```

```
 ServerlessFinance-Compute (Docker image build lại: worker gói kèm wheel Rust)
```

### Benchmark - Trước vs. sau

Lưới 16 tổ hợp SMA, BTCUSDT 1h, cả năm 2025 (≈8.760 nến), cùng memory 1024MB. 16 lần `aws lambda invoke` tuần tự gọi thẳng Worker Lambda (bỏ qua Step Functions để cô lập đúng thời gian thực thi của worker), sau đó query CloudWatch Logs Insights trên các dòng REPORT:

```
fields @message | filter @message like /REPORT/
| parse @message "Billed Duration: * ms" as billed
| stats count(*) as n, avg(billed) as avgMs, max(billed) as maxMs, min(billed) as minMs
```

| | Trước (Python thuần) | Sau (byte-range + cache + Rust) |
|---|---|---|
| n | 16 | 16 |
| Billed Duration trung bình | 419,4375 ms | 44,375 ms |
| Min / Max | 391 / 467 ms | 34 / 58 ms |

**→ Nhanh gấp ~9,4 lần, giảm ~89% chi phí Lambda** cho function được gọi nhiều nhất hệ thống. Kết quả được kiểm tra khớp số học với output trước khi đổi, cùng tham số: `total_return`/`max_drawdown`/`win_rate`/`num_trades` khớp tuyệt đối; `sharpe_ratio` lệch ở chữ số thập phân thứ ~13, nhiễu làm tròn dấu phẩy động thông thường, không phải lỗi tính toán.

{{% notice warning %}}
Trong lúc benchmark, em cũng phát hiện quota concurrent-execution mặc định của Lambda trên tài khoản AWS này chỉ là **10**, thấp hơn `max_concurrency=50` đã cấu hình cho Distributed Map (Phase 3). Một job đủ lớn có thể gặp `Lambda.TooManyRequestsException`. Đã ghi lại cách khắc phục là xin tăng quota hoặc đổi `max_concurrency` một dòng, chỉ áp dụng khi thực sự cần.
{{% /notice %}}

Tiếp tục tới [Phase 7 - Observability & Hardening](../5.9-Observability/).
