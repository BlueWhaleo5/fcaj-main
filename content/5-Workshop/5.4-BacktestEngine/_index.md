---
title: "Phase 2 - Backtesting Engine Core"
date: 2026-07-30
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
includeInReport: true
---

### Objective

Build the backtest simulator as a pure-Python module, includes a strategy interface, a bar-by-bar equity-curve simulator, and summary metrics (Sharpe ratio, max drawdown, win rate). This is deliberately tested and validated locally *before* it's wired into any distributed compute (Phase 3), so that when the Worker Lambda later runs it thousands of times in parallel, its correctness is already established against known-good numbers.

### Strategy interface - `strategies/base.py`

```python
class Strategy:
    def __init__(self, **params):
        self.params = params

    def generate_signals(self, df: pd.DataFrame) -> pd.Series:
        """Given OHLCV data (must have a 'close' column), return a Series aligned to
        df.index with the target position at the close of each bar: 1 (long) or 0 (flat).
        """
        raise NotImplementedError
```

### Example strategy - `strategies/sma_crossover.py`

```python
class SmaCrossoverStrategy(Strategy):
    """Long-only: hold when the fast SMA is above the slow SMA, flat otherwise."""

    def generate_signals(self, df: pd.DataFrame) -> pd.Series:
        fast = df["close"].rolling(self.fast_period).mean()
        slow = df["close"].rolling(self.slow_period).mean()
        signal = pd.Series(0, index=df.index, dtype=int)
        signal[fast > slow] = 1
        return signal
```

### Simulator core - `services/common/engine.py`

The execution model shifts a signal by one bar before acting on it, to avoid lookahead bias, a signal computed from bar *i*'s close is first actionable at bar *i+1*:

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

### Verify - run locally, no AWS credentials needed

```bash
pip install -r requirements.txt
pytest
```

```
....                                                                     [100%]
4 passed in 3.46s
```

The test suite checks the simulator's output against hand-computed values for a fixed synthetic OHLCV series, the same numeric baseline used later in Phase 6 to confirm the Rust-accelerated version produces identical results.

Continue to [Phase 3 - Distributed Compute](../5.5-DistributedCompute/).
