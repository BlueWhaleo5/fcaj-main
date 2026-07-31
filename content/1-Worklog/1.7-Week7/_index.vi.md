---
title: "Worklog Tuần 7"
date: 2026-07-30
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
includeInReport: true
reportTableColumns:
  - Thứ
  - Công việc
  - Ngày hoàn thành
reportHeadings:
  - Mục tiêu tuần 7
  - Các công việc cần triển khai trong tuần này
  - Kết quả đạt được tuần 7
reportType: worklog
---

### Mục tiêu tuần 7:

* Deploy Phase 4 (lớp API) và kiểm chứng bằng `curl`.
* Bắt đầu Phase 5 - thiết kế module paper trading thời gian thực và lớp dữ liệu.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 1 | `cdk deploy ServerlessFinance-Api`; kiểm chứng tạo job/trạng thái/kết quả/403/404 bằng `curl` | 15/06/2025 | 15/06/2025 | |
| 2 | Thiết kế data model paper trading: account, vị thế, lệnh, số dư tiền mặt | 16/06/2025 | 16/06/2025 | |
| 3 | So sánh ElastiCache vs. DynamoDB cho trạng thái này. Quyết định dùng DynamoDB on-demand vì chi phí gần $0 lúc rảnh | 17/06/2025 | 17/06/2025 | |
| 4 | Hiện thực khớp lệnh: MARKET khớp ngay theo giá thực, LIMIT chỉ khớp nếu đã cắt qua | 18/06/2025 | 18/06/2025 | Binance API docs (`/ticker/price`) |
| 5 | Hiện thực logic cập nhật cash/vị thế với phí 0,1%, khớp giả định của engine ở Phase 2 | 19/06/2025 | 19/06/2025 | |

### Kết quả đạt được tuần 7:

* Phase 4 đã deploy và kiểm chứng: gọi API thật trả đúng `201`/`200`/`403`/`404` như kỳ vọng.
* Đã đưa ra và ghi lại quyết định kiến trúc quan trọng: ElastiCache → DynamoDB on-demand cho trạng thái paper trading (do chi phí) (xem [Bản đề xuất §4](../../2-Proposal/#4-triển-khai-kỹ-thuật)).
* Đã hiện thực logic khớp lệnh và cập nhật fill trong `services/trading/handler.py`, chưa deploy.
