---
title: "Worklog Tuần 4"
date: 2026-07-30
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
includeInReport: true
reportTableColumns:
  - Thứ
  - Công việc
  - Ngày hoàn thành
reportHeadings:
  - Mục tiêu tuần 4
  - Các công việc cần triển khai trong tuần này
  - Kết quả đạt được tuần 4
reportType: worklog
---

### Mục tiêu tuần 4:

* Xây dựng Phase 2 - lõi engine backtest, hoàn toàn cục bộ, không phụ thuộc AWS.
* Unit test khớp với số liệu tính tay trước khi gắn vào bất kỳ compute phân tán nào.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 1 | Thiết kế interface `Strategy` và chiến lược đầu tiên (SMA crossover) | 25/05/2026 | 26/05/2026 | |
| 2 | Hiện thực simulator equity-curve theo từng nến với mô hình thực thi dịch một nến (tránh lookahead bias) | 26/05/2026 | 26/05/2026 | |
| 3 | Hiện thực các metric tổng hợp: Sharpe ratio, max drawdown, win rate, trích xuất trade | 27/05/2026 | 27/05/2026 | |
| 4 | Viết unit test trên chuỗi OHLCV giả lập cố định, so với giá trị kỳ vọng tính tay | 28/05/2026 | 29/05/2026 | pytest docs |
| 5 | Chạy `pytest` xanh cục bộ, không cần AWS credentials | 29/05/2026 | 29/05/2026 | |

### Kết quả đạt được tuần 4:

* Hoàn tất Phase 2: `services/common/engine.py` + `strategies/sma_crossover.py`, đã unit test.
* `pytest` chạy pass cục bộ trong vài giây, baseline này được dùng để đối chiếu số học với lõi Rust ở Phase 6 sau này.
* Chi tiết đầy đủ: [Workshop - Phase 2 Lõi Engine Backtest](../../5-Workshop/5.4-BacktestEngine/).
