---
title: "Tổng quan Workshop"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
includeInReport: true
---

### Chúng ta sẽ xây dựng gì

**ServerlessFinance** là nền tảng AWS serverless gồm hai module độc lập, dùng chung dữ liệu giá lịch sử:

1. **Engine backtest**: ingest dữ liệu nến OHLCV lịch sử cho một cặp crypto, sau đó chạy một chiến lược giao dịch trên lưới lớn các tổ hợp tham số (ví dụ mọi tổ hợp chu kỳ moving-average nhanh/chậm) song song, xếp hạng kết quả theo Sharpe ratio.
2. **Paper trading thời gian thực**: đặt lệnh market/limit giả lập khớp theo giá thực, theo dõi tiền mặt và vị thế, đẩy thông báo khớp lệnh tới client đang kết nối qua WebSocket.

Cả hai đều chạy mà không cần một server, container hay database instance nào chạy nền, mọi thứ đều tính phí theo request (Lambda, API Gateway, Step Functions) hoặc lưu trữ on-demand (S3, DynamoDB).

![Kiến trúc ServerlessFinance](/images/5-Workshop/serverless_architecture.png)

### Cấu trúc repo

```
ServerlessFinance/
├── infra/                  # AWS CDK app (Python) — 5 stack độc lập
│   ├── app.py
│   └── stacks/
│       ├── data_lake_stack.py       # Phase 1
│       ├── compute_stack.py         # Phase 3
│       ├── api_stack.py             # Phase 4
│       ├── realtime_stack.py        # Phase 5
│       └── observability_stack.py   # Phase 7
├── services/                # Lambda handlers
│   ├── ingestion/ orchestrator/ worker/ aggregator/ api/
│   ├── trading/ trading_ws_connect/ trading_ws_disconnect/
│   └── common/               # engine dùng chung, data access, wrapper Rust
├── strategies/               # các chiến lược có thể cắm thêm
├── rust/backtest_core/        # lõi Rust/PyO3 tăng tốc (Phase 6)
└── tests/                    # unit test cục bộ, không cần AWS
```

### Cách workshop này được tổ chức

Mỗi trang dưới đây tương ứng với một phase trong quá trình xây dựng: đã được implement, deploy bằng `cdk deploy` lên tài khoản AWS, và kiểm chứng trên hạ tầng AWS trước khi sang phase kế tiếp.

Sau khi hoàn thành, chúng ta sẽ có:
- Một data lake hoạt động, ingest dữ liệu thị trường Binance thật.
- Một engine backtest phân tán có thể xếp hạng hàng trăm tổ hợp tham số song song.
- Một REST API và một module paper trading thời gian thực với kênh đẩy WebSocket.
- Một worker tăng tốc bằng Rust, nhanh hơn ~9,4 lần so với bản Python thuần.
- Một dashboard và alarm CloudWatch theo dõi toàn bộ hệ thống.

Tiếp tục tới [Yêu cầu chuẩn bị](../5.2-Prerequisites/) để thiết lập môi trường.
