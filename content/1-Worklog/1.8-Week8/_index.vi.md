---
title: "Worklog Tuần 8"
date: 2026-07-30
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
includeInReport: true
reportTableColumns:
  - Thứ
  - Công việc
  - Ngày hoàn thành
reportHeadings:
  - Mục tiêu tuần 8
  - Các công việc cần triển khai trong tuần này
  - Kết quả đạt được tuần 8
reportType: worklog
---

### Mục tiêu tuần 8:

* Thêm kênh đẩy WebSocket cho thông báo khớp lệnh thời gian thực.
* Deploy Phase 5 và kiểm chứng toàn bộ luồng paper trading, gồm cả client WebSocket thật.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 1 | Thiết kế WebSocket API: route `$connect`/`$disconnect`, bảng connections theo `connectionId` | 22/06/2026 | 23/06/2026 | AWS API Gateway WebSocket docs |
| 2 | Hiện thực Lambda `trading_ws_connect`/`trading_ws_disconnect` | 23/06/2026 | 24/06/2026 | |
| 3 | Nối Trading Lambda để đẩy fill qua `apigatewaymanagementapi.post_to_connection` | 24/06/2026 | 25/06/2026 | |
| 4 | Lần deploy đầu: gặp lỗi `cdk deploy` → Amazon Timestream không có mặt trên CloudFormation ở `ap-southeast-1` | 25/06/2026 | 25/06/2026 | |
| 5 | Thiết kế lại lưu trữ lịch sử P&L dùng DynamoDB thay Timestream → deploy lại thành công | 26/06/2026 | 26/06/2026 | |
| 6 | Kiểm chứng đầu-cuối: tạo account, đặt lệnh, quan sát client kiểu `wscat` nhận fill theo thời gian thực | 27/06/2026 | 27/06/2026 | |

### Kết quả đạt được tuần 8:

* Phase 5 đã deploy và kiểm chứng: khớp lệnh, số học cash/vị thế, và đẩy WebSocket thời gian thực đều xác nhận trên account.
* Rút kinh nghiệm: Amazon Timestream for LiveAnalytics không khả dụng ở `ap-southeast-1` → cách khắc phục (DynamoDB) giờ là mẫu chuẩn cho kiểu lịch sử P&L trong dự án này.
* Chi tiết đầy đủ: [Workshop - Phase 5](../../5-Workshop/5.7-PaperTrading/).
