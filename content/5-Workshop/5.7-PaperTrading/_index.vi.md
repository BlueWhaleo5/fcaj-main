---
title: "Phase 5 - Paper Trading Thời gian thực"
date: 2026-07-30
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
includeInReport: true
---

### Mục tiêu

Một module thứ hai, độc lập: đặt lệnh market/limit giả lập khớp theo giá Binance **thực**, theo dõi số dư tiền mặt và vị thế trong DynamoDB, và đẩy thông báo khớp lệnh tới mọi client đang kết nối theo thời gian thực qua WebSocket.

### Các quyết định thiết kế thay đổi trong lúc xây dựng trên AWS thật

{{% notice info %}}
Thiết kế ban đầu dùng **ElastiCache** (vị thế/lệnh) và **Amazon Timestream** (lịch sử P&L), với **AppSync** cho đẩy thời gian thực. Cả hai đều bị thay thế sau khi gặp giới hạn:
- **ElastiCache → DynamoDB on-demand**: ElastiCache Serverless tính phí storage + ECPU kể cả khi không có traffic; DynamoDB on-demand gần như $0 lúc rảnh.
- **Timestream → DynamoDB**: một lần `cdk deploy` bị lỗi `Unrecognized resource types: [AWS::Timestream::Table, AWS::Timestream::Database]`. Timestream for LiveAnalytics không có mặt ở `ap-southeast-1`.
- **AppSync → API Gateway WebSocket**: plan ban đầu coi hai lựa chọn này tương đương; WebSocket API không cần GraphQL schema.
{{% /notice %}}

### Kiến trúc

![Kiến trúc Phase 5 - Paper Trading Thời gian thực](/images/5-Workshop/phase5_paper_trading.png)

### Khớp lệnh - `services/trading/handler.py`

Lệnh MARKET khớp ngay theo giá Binance thực; lệnh LIMIT khớp ngay nếu đã cắt qua, ngược lại giữ trạng thái `OPEN`:

```python
def _place_order(account_id, body):
    price = _latest_price(symbol)   # GET https://api.binance.com/api/v3/ticker/price

    crossed = order_type == "MARKET" or (
        (side == "BUY" and price <= limit_price) or (side == "SELL" and price >= limit_price)
    )
    fill_price = limit_price if (order_type == "LIMIT" and crossed) else price
    status = "FILLED" if crossed else "OPEN"

    if status == "FILLED":
        fill = _apply_fill(account_id, symbol, side, qty, fill_price)   # cập nhật cash + vị thế
        _write_pnl_point(account_id, symbol, fill["cashBalance"], fill["qty"], position_value)
        _push_to_connections(account_id, {"type": "ORDER_FILLED", "order": order_item, ...})
```

### Deploy

```bash
cd infra
cdk deploy ServerlessFinance-Realtime --require-approval never
```

```
 ServerlessFinance-Realtime
Outputs:
ServerlessFinance-Realtime.PaperTradingApiUrl = https://5zn6uw7ta0.execute-api.ap-southeast-1.amazonaws.com/prod/
ServerlessFinance-Realtime.PaperTradingWsUrl = wss://y52nuv3x41.execute-api.ap-southeast-1.amazonaws.com/prod
```

### Kiểm chứng - Đặt lệnh, quan sát push qua WebSocket

```bash
$ aws apigateway test-invoke-method --rest-api-id 5zn6uw7ta0 --resource-id <accountsResourceId> \
    --http-method POST --body '{"initialCash": 50000}'
```

```json
{"status": 201, "body": "{\"accountId\": \"d7445402-87cf-4341-8fc5-57efdb602c2a\", \"cashBalance\": 50000}"}
```

```bash
$ aws apigateway test-invoke-method --rest-api-id 5zn6uw7ta0 --resource-id <ordersResourceId> \
    --path-with-query-string "/paper/accounts/d7445402-87cf-4341-8fc5-57efdb602c2a/orders" \
    --http-method POST --body '{"symbol": "BTCUSDT", "side": "BUY", "type": "MARKET", "qty": 0.01}'
```

```json
{"status": 201, "body": "{\"orderId\": \"6318a859-...\", \"symbol\": \"BTCUSDT\", \"side\": \"BUY\", \"status\": \"FILLED\", \"filledPrice\": 64042.01}"}
```

Một client WebSocket đã kết nối tới `wss://y52nuv3x41.execute-api.ap-southeast-1.amazonaws.com/prod?accountId=d7445402-87cf-4341-8fc5-57efdb602c2a` từ trước, khi lệnh thứ hai (SELL) được đặt, nhận được ngay theo thời gian thực:

```json
{
  "type": "ORDER_FILLED",
  "order": {"orderId": "228ab206-...", "symbol": "BTCUSDT", "side": "SELL", "qty": 0.004,
             "status": "FILLED", "filledPrice": 64079.98},
  "cashBalance": 49615.00307998,
  "position": {"symbol": "BTCUSDT", "qty": 0.006, "avgEntryPrice": 64042.01}
}
```

Số học cash khớp đúng: mua 0,01 BTC @ $64.042,01 (chi phí $641,06 gồm phí 0,1%) rồi bán 0,004 BTC @ $64.079,98 (thu về $256,06 sau phí) → cash `50000 − 641,0605 + 256,0636 = 49615,00307998`, vị thế `0,01 − 0,004 = 0,006` BTC, khớp đúng với push thời gian thực, đến chữ số thập phân thứ 8.

Tiếp tục tới [Phase 6 - Tối ưu Chi phí & Hiệu năng](../5.8-CostPerformance/).
