---
title: "Phase 2 - Lõi Engine Backtest"
date: 2026-07-30
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
includeInReport: true
---

### Mục tiêu

Xây dựng simulator backtest thành một module Python thuần, gồm interface chiến lược, simulator tính equity curve theo từng nến, và các metric tổng hợp (Sharpe ratio, max drawdown, win rate). Phần này được test và kiểm chứng cục bộ *trước khi* gắn vào bất kỳ compute phân tán nào (Phase 3), để khi Worker Lambda sau này chạy nó hàng nghìn lần song song, tính đúng đắn của nó đã được xác lập từ trước với các con số đã biết là đúng.

### Interface chiến lược - `strategies/base.py`

```python
class Strategy:
    def __init__(self, **params):
        self.params = params

    def generate_signals(self, df: pd.DataFrame) -> pd.Series:
        """Nhận dữ liệu OHLCV (phải có cột 'close'), trả về Series khớp với
        df.index thể hiện vị thế mục tiêu tại close mỗi nến: 1 (long) hoặc 0 (flat).
        """
        raise NotImplementedError
```

### Chiến lược ví dụ - `strategies/sma_crossover.py`

```python
class SmaCrossoverStrategy(Strategy):
    """Chỉ long: giữ vị thế khi SMA nhanh > SMA chậm, ngược lại thoát vị thế."""

    def generate_signals(self, df: pd.DataFrame) -> pd.Series:
        fast = df["close"].rolling(self.fast_period).mean()
        slow = df["close"].rolling(self.slow_period).mean()
        signal = pd.Series(0, index=df.index, dtype=int)
        signal[fast > slow] = 1
        return signal
```

### Lõi simulator - `services/common/engine.py`

Mô hình thực thi dịch tín hiệu đi một nến trước khi hành động, để tránh lookahead bias, tín hiệu tính từ close của nến *i* chỉ có thể hành động sớm nhất ở nến *i+1*:

```python
def run_backtest(df, strategy, config=None):
    config = config or BacktestConfig()
    signals = strategy.generate_signals(df).fillna(0)
    executed_position = signals.shift(1).fillna(0)

    bar_returns = df["close"].pct_change().fillna(0)
    strategy_returns = executed_position * bar_returns

    position_change = executed_position.diff()
    position_change.iloc[0] = executed_position.iloc[0]
    fees = position_change.abs() * config.fee_rate

    net_returns = strategy_returns - fees
    equity_curve = (config.initial_capital * (1 + net_returns).cumprod())

    trades = _extract_trades(df, executed_position, position_change)
    return BacktestResult(
        equity_curve=equity_curve,
        trades=trades,
        total_return=float(equity_curve.iloc[-1] / config.initial_capital - 1),
        sharpe_ratio=_sharpe_ratio(net_returns, config.periods_per_year),
        max_drawdown=_max_drawdown(equity_curve),
        win_rate=(len([t for t in trades if t.is_win]) / len(trades)) if trades else float("nan"),
    )
```

### Kiểm chứng - chạy cục bộ, không cần AWS credentials

```bash
pip install -r requirements.txt
pytest
```

```
....                                                                     [100%]
4 passed in 3.46s
```

Bộ test kiểm tra output của simulator so với các giá trị tính tay trên một chuỗi OHLCV giả lập cố định, cùng baseline số học này được dùng lại ở Phase 6 để xác nhận bản tăng tốc bằng Rust cho ra kết quả giống hệt.

Tiếp tục tới [Phase 3 - Compute Phân tán](../5.5-DistributedCompute/).
