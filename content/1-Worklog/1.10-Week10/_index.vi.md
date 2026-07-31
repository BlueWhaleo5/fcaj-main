---
title: "Worklog Tuần 10"
date: 2026-07-30
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
includeInReport: true
reportTableColumns:
  - Thứ
  - Công việc
  - Ngày hoàn thành
reportHeadings:
  - Mục tiêu tuần 10
  - Các công việc cần triển khai trong tuần này
  - Kết quả đạt được tuần 10
reportType: worklog
---

### Mục tiêu tuần 10:

* Học đủ Rust + PyO3 để port phần tính metrics của engine backtest.
* Build bên trong Docker multi-stage image (không cần cài Rust cục bộ), deploy, và benchmark lại.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 1 | Học cơ bản PyO3: `#[pyfunction]`, `#[pymodule]`, build wheel bằng `maturin` | 06/07/2026 | 06/07/2026 | PyO3 / maturin docs |
| 2 | Port trung thành phần toán equity-curve/Sharpe/drawdown/trích xuất trade của `engine.py` sang Rust (kể cả một quirk index âm có sẵn) | 07/07/2026 | 08/07/2026 | |
| 3 | Viết `Dockerfile` multi-stage: stage `rust-builder` (yum install gcc, rustup, maturin build) đưa vào image Lambda cuối | 08/07/2026 | 08/07/2026 | |
| 4 | Debug package manager của base image (base image Lambda này không có `dnf` mà là `yum`) | 09/07/2026 | 10/07/2026 | |
| 5 | Deploy, smoke-test một tổ hợp tham số để kiểm tra khớp số học với output trước khi đổi, rồi chạy lại benchmark 16 tổ hợp | 11/07/2026 | 11/07/2026 | |

### Kết quả đạt được tuần 10:

* Lõi Rust/PyO3 (`rust/backtest_core`) đã build và deploy bên trong Docker image của worker, không cần cross-compile từ máy dev.
* **Đo được tốc độ tăng 9,4 lần trên AWS thật**: 419,4 ms → 44,4 ms Billed Duration trung bình, giảm ~89% chi phí compute Lambda.
* Xác nhận khớp số học với output Python trước khi đổi, cùng tham số.
* Chi tiết đầy đủ kèm Dockerfile và số benchmark: [Workshop - Phase 6](../../5-Workshop/5.8-CostPerformance/).
