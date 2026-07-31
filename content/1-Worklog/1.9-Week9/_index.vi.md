---
title: "Worklog Tuần 9"
date: 2026-07-30
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
includeInReport: true
reportTableColumns:
  - Thứ
  - Công việc
  - Ngày hoàn thành
reportHeadings:
  - Mục tiêu tuần 9
  - Các công việc cần triển khai trong tuần này
  - Kết quả đạt được tuần 9
reportType: worklog
---

### Mục tiêu tuần 9:

* Bắt đầu Phase 6 - tối ưu chi phí & hiệu năng hot path của Worker Lambda.
* Đo baseline hiệu năng trước, sau đó hiện thực S3 byte-range fetch và cache `/tmp`.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 1 | Benchmark worker hiện tại (Python thuần): job 16 tổ hợp, lấy Billed Duration từ CloudWatch | 29/06/2025 | 29/06/2025 | |
| 2 | Tìm hiểu vì sao `s3.get_object` tải nguyên file Parquet; học column projection của `pyarrow.fs.S3FileSystem` | 30/06/2025 | 01/07/2025 | pyarrow docs |
| 3 | Viết lại `data_access.py` để chỉ lấy đúng cột cần qua HTTP Range GET thật | 01/07/2025 | 01/07/2025 | |
| 4 | Thêm cache `/tmp` theo key bucket + key + columns, tận dụng việc Lambda tái sử dụng warm-container| 02/07/2025 | 02/07/2025 | |
| 5 | Cập nhật Worker Lambda chỉ yêu cầu `["open_time", "close"]` | 03/07/2025 | 03/07/2025 | |

### Kết quả đạt được tuần 9:

* Kết quả đo baseline: trung bình 419,4 ms Billed Duration trên 16 lần gọi worker tuần tự.
* Đã hiện thực S3 byte-range fetch + cache `/tmp`.
* Sẵn sàng thêm lõi Rust tuần sau và đo lại.
