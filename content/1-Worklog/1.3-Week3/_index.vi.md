---
title: "Worklog Tuần 3"
date: 2026-07-30
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
includeInReport: true
reportTableColumns:
  - Thứ
  - Công việc
  - Ngày hoàn thành
reportHeadings:
  - Mục tiêu tuần 3
  - Các công việc cần triển khai trong tuần này
  - Kết quả đạt được tuần 3
reportType: worklog
---

### Mục tiêu tuần 3:

* Xây dựng Phase 1 - Data Lake: ingest dữ liệu OHLCV từ Binance, lưu Parquet có partition trên S3.
* Dựng catalog Athena/Glue để query dữ liệu đã ingest mà không cần crawler.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 1 | Tìm hiểu REST API công khai của Binance (`/api/v3/klines`) và thiết kế cấu trúc partition S3 | 18/05/2026 | 18/05/2026 | Binance API docs |
| 2 | Viết Ingestion Lambda (lấy klines, chuyển sang Parquet bằng pandas/pyarrow) | 18/05/2026 | 18/05/2026 | |
| 3 | Tìm hiểu vì sao pandas + pyarrow cần container-image Lambda (giới hạn zip 250MB); viết Dockerfile | 19/05/2026 | 20/05/2026 | AWS Lambda docs |
| 4 | Định nghĩa `DataLakeStack` trong CDK: S3 bucket, DockerImageFunction, lịch EventBridge hàng ngày | 20/05/2026 | 21/05/2026 | |
| 5 | Thêm bảng Glue Catalog dùng partition projection; deploy và backfill một năm dữ liệu BTCUSDT | 22/05/2026 | 22/05/2026 | |
| 6 | Kiểm chứng bằng Athena: `SELECT COUNT(*)` theo từng partition khớp số dòng kỳ vọng | 23/05/2026 | 23/05/2026 | |

### Kết quả đạt được tuần 3:

* Phase 1 đã deploy và kiểm chứng trên AWS: `cdk deploy ServerlessFinance-DataLake` chạy thành công, Ingestion Lambda backfill 12 tháng dữ liệu BTCUSDT 1h.
* Query Athena trên bảng dùng partition projection (không crawler) trả về đúng số dòng (720 dòng cho một tháng 30 ngày, khung 1h).
* Chi tiết đầy đủ: [Workshop - Phase 1 Data Lake](../../5-Workshop/5.3-DataLake/).
