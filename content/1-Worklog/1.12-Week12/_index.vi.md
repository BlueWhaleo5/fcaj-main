---
title: "Worklog Tuần 12"
date: 2026-07-30
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
includeInReport: true
reportTableColumns:
  - Thứ
  - Công việc
  - Ngày hoàn thành
reportHeadings:
  - Mục tiêu tuần 12
  - Các công việc cần triển khai trong tuần này
  - Kết quả đạt được tuần 12
reportType: worklog
---

### Mục tiêu tuần 12:

* Kiểm chứng toàn hệ thống lần cuối trên cả 8 phase trên AWS thật, sau đó tháo dỡ hết để không phát sinh chi phí.
* Viết dự án thành báo cáo thực tập FCAJ này (Proposal + Workshop + Worklog).

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 1 | `cdk deploy --all`; xử lý sự cố Docker Desktop engine giữa chừng, deploy lại sạch | 20/07/2025 | 20/07/2025 | |
| 2 | Chạy lại kiểm chứng đầu-cuối từng phase: ingest dữ liệu, chạy job Step Functions, gọi REST API, paper-trade với push WebSocket, kiểm tra cả 8 alarm | 21/07/2025 | 21/07/2025 | |
| 3 | `cdk destroy --all`, sau đó xóa thủ công S3 bucket và bảng DynamoDB có `RemovalPolicy.RETAIN`, xác nhận account về 0 resource ServerlessFinance | 22/07/2025 | 22/07/2025 | |
| 4 | Viết phần Proposal (kiến trúc, dịch vụ AWS, phân tích chi phí, đánh giá rủi ro) | 23/07/2025 | 23/07/2025 | |
| 5 | Viết phần Workshop | 24/07/2025 | 24/07/2025 | |
| 6 | Viết Worklog này, điền phần Tự đánh giá và Feedback | 25/07/2025 | 25/07/2025 | |

### Kết quả đạt được tuần 12:

* Xác nhận mọi phase vẫn hoạt động đầu-cuối sau 12 tuần thay đổi tăng dần.
* Hoàn thành báo cáo thực tập
