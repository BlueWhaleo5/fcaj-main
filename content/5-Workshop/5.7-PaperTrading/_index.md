---
title: "Phase 5 - Real-time Paper Trading"
date: 2026-07-30
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
includeInReport: true
---

### Objective

A second, independent module: place simulated market/limit orders that fill against **live** Binance prices, track cash balance and positions in DynamoDB, and push fill notifications to any connected client in real time over a WebSocket.

### Design decisions that changed while building on real AWS

{{% notice info %}}
The original design called for **ElastiCache** (positions/orders) and **Amazon Timestream** (P&L history), with **AppSync** for real-time push. Both were replaced after hitting constraints:
- **ElastiCache → DynamoDB on-demand**: ElastiCache Serverless bills for storage + ECPU even at zero traffic; DynamoDB on-demand is ~$0 idle.
- **Timestream → DynamoDB**: a `cdk deploy` failed with `Unrecognized resource types: [AWS::Timestream::Table, AWS::Timestream::Database]`. Timestream for LiveAnalytics has no presence in `ap-southeast-1`.
- **AppSync → API Gateway WebSocket**: the plan treated these as equivalent; WebSocket API needed no GraphQL schema.
{{% /notice %}}

### Architecture

![Phase 5 - Real-time Paper Trading architecture](/images/5-Workshop/phase5_paper_trading.png)

### Order matching - `services/trading/handler.py`

Market orders fill immediately at the live Binance price; limit orders fill immediately if already crossed, otherwise stay `OPEN`:

```python
def _place_order(account_id, body):
    price = _latest_price(symbol)   # GET https://api.binance.com/api/v3/ticker/price

    crossed = order_type == "MARKET" or (
        (side == "BUY" and price <= limit_price) or (side == "SELL" and price >= limit_price)
    )
    fill_price = limit_price if (order_type == "LIMIT" and crossed) else price
    status = "FILLED" if crossed else "OPEN"

    if status == "FILLED":
        fill = _apply_fill(account_id, symbol, side, qty, fill_price)   # updates cash + position
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

### Verify - Place an order, watch the WebSocket push

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

A WebSocket client connected to `wss://y52nuv3x41.execute-api.ap-southeast-1.amazonaws.com/prod?accountId=d7445402-87cf-4341-8fc5-57efdb602c2a` before a second (SELL) order was placed received, in real time:

```json
{
  "type": "ORDER_FILLED",
  "order": {"orderId": "228ab206-...", "symbol": "BTCUSDT", "side": "SELL", "qty": 0.004,
             "status": "FILLED", "filledPrice": 64079.98},
  "cashBalance": 49615.00307998,
  "position": {"symbol": "BTCUSDT", "qty": 0.006, "avgEntryPrice": 64042.01}
}
```

Cash math checks out exactly: bought 0.01 BTC @ $64,042.01 (cost $641.06 incl. 0.1% fee) then sold 0.004 BTC @ $64,079.98 (proceeds $256.06 after fee) → cash `50000 − 641.0605 + 256.0636 = 49615.00307998`, position `0.01 − 0.004 = 0.006` BTC, matching the live push exactly, to the 8th decimal.

Continue to [Phase 6 - Cost & Performance Optimization](../5.8-CostPerformance/).
