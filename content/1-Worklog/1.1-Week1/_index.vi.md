---
title: "Worklog Tuần 1"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
includeInReport: true
reportTableColumns:
  - Thứ
  - Công việc
  - Ngày hoàn thành
reportHeadings:
  - Mục tiêu tuần 1
  - Các công việc cần triển khai trong tuần này
  - Kết quả đạt được tuần 1
reportType: worklog
---

### Mục tiêu tuần 1:

* Làm quen với chương trình FCAJ, mentor, và AWS Console/CLI.
* Hiểu các nhóm dịch vụ AWS cốt lõi (Compute, Storage, Networking, Database) trước khi chọn dự án.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 1 | Onboarding: nội quy chương trình, gặp mentor và các bạn thực tập khác | 01/05/2026 | 01/05/2026 | |
| 2 | Tìm hiểu các nhóm dịch vụ AWS (Compute, Storage, Networking, Database, Serverless) | 02/05/2026 | 04/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | Tạo AWS account, cài & cấu hình AWS CLI, tìm hiểu IAM cơ bản | 05/05/2026 | 05/05/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | Tìm hiểu các dịch vụ serverless liên quan tới ý tưởng dự án: Lambda, S3, DynamoDB, API Gateway | 06/05/2026 | 07/05/2026 | Tài liệu AWS |
| 5 | Phác thảo ý tưởng cho **ServerlessFinance** (engine backtest serverless) và kiểm tra với AWS Free Tier / Pricing Calculator | 08/05/2026 | 08/05/2026 | |

### Kết quả đạt được tuần 1:

* Có AWS account hoạt động, AWS CLI đã cấu hình (xác nhận bằng `aws sts get-caller-identity`).
* Hình dung rõ những dịch vụ AWS cần cho một pipeline dữ liệu + compute hoàn toàn serverless: S3, Lambda, Step Functions, DynamoDB, API Gateway.
* Chốt hướng dự án: engine backtest chiến lược phân tán kèm module paper trading thời gian thực, cả hai đều serverless (xem [Bản đề xuất](../../2-Proposal/)).
