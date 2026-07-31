---
title: "Worklog Tuần 5"
date: 2026-07-30
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
includeInReport: true
reportTableColumns:
  - Thứ
  - Công việc
  - Ngày hoàn thành
reportHeadings:
  - Mục tiêu tuần 5
  - Các công việc cần triển khai trong tuần này
  - Kết quả đạt được tuần 5
reportType: worklog
---

### Mục tiêu tuần 5:

* Bắt đầu Phase 3 - biến engine backtest cục bộ thành job phân tán bằng Step Functions Distributed Map.
* Hiện thực Worker Lambda và Orchestrator Lambda.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 1 | Tìm hiểu Step Functions Distributed Map: `items_path`, `item_selector`, `max_concurrency`, `ResultWriter` | 01/06/2026 | 01/06/2026 | AWS Step Functions docs |
| 2 | Viết Worker Lambda: đọc OHLCV từ S3, chạy `engine.py`, trả về metric tổng hợp | 02/06/2026 | 02/06/2026 | |
| 3 | Viết Orchestrator Lambda: mở rộng lưới tham số thành danh sách item cho Distributed Map, khởi chạy execution | 03/06/2026 | 03/06/2026 | |
| 4 | Thiết kế schema DynamoDB cho job/kết quả (`BacktestJobsTable`, `BacktestResultsTable`) | 04/06/2026 | 04/06/2026 | |
| 5 | Gắn `add_catch` cho Distributed Map để job lỗi được đánh dấu `FAILED`, không bị treo | 05/06/2026 | 06/06/2026 | |

### Kết quả đạt được tuần 5:

* Hiện thực Worker và Orchestrator Lambda; state machine Distributed Map định nghĩa trong `compute_stack.py`.
* Hiểu vì sao `PythonFunction` dạng zip không dùng được cho worker (kích thước pandas/pyarrow) (tái sử dụng pattern container-image từ Phase 1).
* Sẵn sàng deploy và kiểm chứng vào tuần sau.
