---
title: "Workshop"
date: 2026-07-30
weight: 5
chapter: false
pre: " <b> 5. </b> "
includeInReport: false
---

# Xây dựng ServerlessFinance: Nền tảng Backtest & Paper Trading Serverless trên AWS

#### Tổng quan

Workshop này hướng dẫn xây dựng **ServerlessFinance** đầu-cuối trên AWS: một data lake cho dữ liệu giá crypto lịch sử, một engine backtest phân tán chạy song song lưới tham số chiến lược trên nhiều Lambda worker, một REST API đứng trước nó, một module paper trading thời gian thực với kênh đẩy WebSocket, một phần lõi tăng tốc bằng Rust, và một lớp observability bằng CloudWatch, mỗi phase đều được deploy và kiểm chứng trên AWS trước khi sang phase tiếp theo.

#### Nội dung

1. [Tổng quan Workshop](5.1-Overview/)
2. [Yêu cầu chuẩn bị](5.2-Prerequisites/)
3. [Phase 1 - Data Lake](5.3-DataLake/)
4. [Phase 2 - Lõi Engine Backtest](5.4-BacktestEngine/)
5. [Phase 3 - Compute Phân tán](5.5-DistributedCompute/)
6. [Phase 4 - Lớp API](5.6-APILayer/)
7. [Phase 5 - Paper Trading Thời gian thực](5.7-PaperTrading/)
8. [Phase 6 - Tối ưu Chi phí & Hiệu năng](5.8-CostPerformance/)
9. [Phase 7 - Observability & Hardening](5.9-Observability/)
10. [Dọn dẹp tài nguyên](5.10-Cleanup/)