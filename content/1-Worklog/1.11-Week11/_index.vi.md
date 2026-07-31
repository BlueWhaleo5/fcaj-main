---
title: "Worklog Tuần 11"
date: 2026-07-30
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
includeInReport: true
reportTableColumns:
  - Thứ
  - Công việc
  - Ngày hoàn thành
reportHeadings:
  - Mục tiêu tuần 11
  - Các công việc cần triển khai trong tuần này
  - Kết quả đạt được tuần 11
reportType: worklog
---

### Mục tiêu tuần 11:

* Xây dựng Phase 7 - dashboard và bộ alarm CloudWatch phủ toàn bộ Lambda và state machine.
* Thêm dead-letter queue cho Lambda duy nhất trong hệ thống được gọi bất đồng bộ.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 1 | Rà soát mô hình gọi của từng Lambda (sync API Gateway proxy/Step Functions Task vs. async EventBridge) để tìm cái nào thực sự cần DLQ | 13/07/2025 | 13/07/2025 | |
| 2 | Thêm SQS DLQ + `retry_attempts` cho Ingestion Lambda | 13/07/2025 | 14/07/2025 | |
| 3 | Thiết kế `observability_stack.py`: đọc metric theo tên/ARN function từ 4 stack còn lại, không ràng buộc IAM chiều nào | 14/07/2025 | 15/07/2025 | |
| 4 | Dựng dashboard CloudWatch (lỗi/duration mỗi Lambda, execution Step Functions) và 8 alarm, nối tới SNS topic | 15/07/2025 | 17/07/2025 | AWS CloudWatch docs |
| 5 | Deploy, kiểm chứng cả 8 alarm báo `OK`, xác nhận/xử lý subscription email SNS | 17/07/2025 | 18/07/2025 | |

### Kết quả đạt được tuần 11:

* Phase 7 đã deploy và kiểm chứng: cả 8 alarm `OK`, dashboard hiển thị metric thật, SQS DLQ đã gắn cho Ingestion Lambda.
* **Cả 8 phase của ServerlessFinance đã hoàn thành**
* Chi tiết đầy đủ: [Workshop - Phase 7](../../5-Workshop/5.9-Observability/).
