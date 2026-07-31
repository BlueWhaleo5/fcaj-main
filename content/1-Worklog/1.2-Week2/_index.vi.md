---
title: "Worklog Tuần 2"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
includeInReport: true
reportTableColumns:
  - Thứ
  - Công việc
  - Ngày hoàn thành
reportHeadings:
  - Mục tiêu tuần 2
  - Các công việc cần triển khai trong tuần này
  - Kết quả đạt được tuần 2
reportType: worklog
---

### Mục tiêu tuần 2:

* Học AWS CDK (Python) làm công cụ Infrastructure-as-Code cho toàn dự án.
* Scaffold repo ServerlessFinance (Phase 0) và chạy được `cdk synth` lần đầu.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 1 | Học khái niệm CDK: App, Stack, Construct (L1/L2), synth vs. deploy | 11/05/2026 | 12/05/2026 | AWS CDK docs |
| 2 | `cdk bootstrap` account/region mục tiêu; viết một CDK app đơn giản và deploy thử | 12/05/2026 | 13/05/2026 | AWS CDK docs |
| 3 | Thiết kế cấu trúc repo: `infra/` (CDK), `services/` (Lambda handler), `strategies/`, `tests/` | 13/05/2026 | 14/05/2026 | |
| 4 | Setup virtualenv Python, `requirements.txt`, tách riêng dependency infra và runtime | 14/05/2026 | 14/05/2026 | |
| 5 | Chạy được `cdk synth` ra CloudFormation template hợp lệ (rỗng), hoàn tất Phase 0 | 15/05/2026 | 16/05/2026 | |

### Kết quả đạt được tuần 2:

* Chọn AWS CDK (Python) làm công cụ IaC cho toàn dự án - xem [Bản đề xuất §4](../../2-Proposal/#4-triển-khai-kỹ-thuật).
* Đã có scaffolding repo: `infra/app.py`, `infra/stacks/`, `services/`, `strategies/`, `tests/`.
* `cdk synth` chạy sạch - Phase 0 đã kiểm chứng. Xem [Workshop - Yêu cầu chuẩn bị](../../5-Workshop/5.2-Prerequisites/) để biết chi tiết các bước setup.
