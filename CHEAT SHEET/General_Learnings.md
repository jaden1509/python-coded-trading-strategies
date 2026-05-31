# Learnings

A running log of concepts, gotchas, and mental models picked up while building these strategies.

---

## Python Fundamentals

- **Variables vs strings**: `TICKER` is a variable, `'TICKER'` is the literal text. Don't put quotes around variable names when passing them as arguments.
- **`=` vs `==`**: single equals assigns a value, double equals compares two things (returns True/False).
- **ALL_CAPS naming**: convention for constants — settings that don't change during the script.
- **f-strings**: `f"Return: {variable:.2f}%"` embeds variables in text. `:.2f` rounds to 2 decimals.
- **Methods vs attributes**: methods end in `()` and do an action (`df.dropna()`), attributes don't (`df.shape`).

---

## Pandas

- **DataFrame = Excel sheet with superpowers**: rows are timestamps, columns are OHLCV.
- **Multi-level columns**: yfinance returns columns like `('Close', 'NQ=F')`. Flatten with `df.columns = df.columns.get_level_values(0)`.
- **Vectorized > loops**: pandas operates on entire columns at once. Never loop through rows if a method exists.
- **`.where()` = Excel IF**: `delta.where(delta > 0, 0)` means "keep value where true, replace with 0 elsewhere."
- **`.loc[]` for conditional writes**: `df.loc[df['RSI'] < 30, 'Signal'] = 1` writes to specific rows only.

---

## Trading Logic

- **Look-ahead bias**: using today's close to decide today's trade is impossible in real life. Always `.shift(1)` your signal before applying to returns.
- **Backtest ≠ live**: a backtest simulates P&L mathematically. No real orders. Strategy returns × signal column = P&L.
- **Signal vs Position**:
  - Signal = should I be in or out RIGHT NOW (1 or 0 every bar)
  - Position = did the signal just change? (`+1` entry, `-1` exit, 0 otherwise)
- **Stateless vs stateful strategies**:
  - EMA crossover = stateless (every bar answers itself)
  - RSI mean reversion = stateful (need to remember if already in trade) → use `ffill()` trick

---

## Performance Metrics

- **Total return alone is meaningless** without context on time period and risk.
- **Annualisation method matters**:
  - Always-in strategies (EMA): `mean_return × bars_per_year`
  - Sometimes-in strategies (RSI): `((1 + total_return) ^ (365/days_traded)) - 1`
- **Profit factor < 1.3 = fragile**: costs and slippage will eat that thin edge.
- **Drawdown is the most important risk metric**: 20% return with 40% drawdown is untradeable.

---

## Gotchas

- **Ticker symbols**: yfinance uses `NQ=F` (not `/NQ=F`, not `NQ1!`).
- **Period limits on yfinance**: max 60 days of intraday data (5min, 15min).
- **`auto_adjust=True`**: explicitly pass this to kill the FutureWarning and get clean OHLC.
- **`adjust=False` in `.ewm()`**: matches standard EMA formula used in TradingView/Bloomberg.
- **NaN propagates**: any math on `NaN` returns `NaN`. Always `.dropna()` after `pct_change()` and `.shift()`.

---

## Workflow

- Build in Colab/Jupyter for fast iteration → push clean `.py` to GitHub.
- Config section at the top = single source of truth for all parameters. Change EMA length in one place, not 50.
- Eight-section structure (imports → config → data → indicators → signals → returns → metrics → plot) works for any strategy.
