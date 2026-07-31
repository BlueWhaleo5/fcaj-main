---
title: "Worklog Tuần 6"
date: 2026-07-30
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
includeInReport: true
reportTableColumns:
  - Thứ
  - Công việc
  - Ngày hoàn thành
reportHeadings:
  - Mục tiêu tuần 6
  - Các công việc cần triển khai trong tuần này
  - Kết quả đạt được tuần 6
reportType: worklog
---

### Mục tiêu tuần 6:

* Deploy và kiểm chứng Phase 3: chạy job backtest nhiều tham số qua Step Functions.
* Bắt đầu Phase 4 - REST API đứng trước Orchestrator, bảo vệ bằng API key.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 1 | `cdk deploy ServerlessFinance-Compute`; sửa lỗi IAM permission phát hiện ở lần deploy đầu | 08/06/2026 | 08/06/2026 | |
| 2 | Chạy job backtest, poll execution Step Functions tới `SUCCEEDED`, kiểm tra kết quả xếp hạng trong DynamoDB | 09/06/2026 | 09/06/2026 | |
| 3 | Học cơ bản API Gateway REST API: resource, method, Lambda proxy integration | 10/06/2026 | 11/06/2026 | AWS API Gateway docs |
| 4 | Setup API key + usage plan; nối `POST /backtests` tới Orchestrator | 12/06/2026 | 12/06/2026 | |
| 5 | Viết API Lambda chỉ đọc cho `GET /backtests/{jobId}` và `GET /backtests/{jobId}/results` | 13/06/2026 | 13/06/2026 | |

### Kết quả đạt được tuần 6:

* Phase 3 kiểm chứng đầu-cuối: job chạy qua Distributed Map (cả đường thành công và đường lỗi).
* Stack REST API của Phase 4 đã soạn xong, sẵn sàng deploy/kiểm chứng.
* Chi tiết đầy đủ: [Workshop - Phase 3](../../5-Workshop/5.5-DistributedCompute/), [Workshop - Phase 4](../../5-Workshop/5.6-APILayer/).
