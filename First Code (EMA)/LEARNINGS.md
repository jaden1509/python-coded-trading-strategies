# Algorithmic Trading — Python Learning Journal

> Self-directed study in quantitative finance and systematic strategy development.

------------------

## Entry 001 — Vectorised EMA Crossover Backtest

**Date:** 2026-05-20

**Script:** `ema_crossover_backtest.py`

**Asset:** NQ=F (Nasdaq-100 Futures)

**Timeframe:** 5miin

**Lookback:** 60 days

**Stack:** `yfinance` · `pandas` · `numpy` · `matplotlib`

---

### Strategy Logic

- Fast EMA (9) > Slow EMA (21) → Long (signal = 1)
- Fast EMA < Slow EMA → Flat (signal = 0)
- Long-only · no short side · no stop-loss

---

### Key Concepts Learned

**`ewm(span, adjust=False)`**
Smaller span reacts faster but amplifies noise; larger span is smoother but lags price. `adjust=False` selects the standard recursive EMA formula over the correction-weighted initialisation.

**Signal → Position via `.diff()`**
`.diff()` on a binary signal series produces +1 at entries and −1 at exits — avoids row-by-row comparisons and stays fully vectorised.

**Lookahead Bias — `.shift(1)`**
```python
df['Strategy_Return'] = df['Market_Return'] * df['Signal'].shift(1)
```
The shift enforces that signals are acted on the *following* bar. Without it, today's signal trades today's return — a form of data snooping that inflates backtest performance.

**Equity Curve via Compounding**
```python
df['Equity'] = (1 + df['Strategy_Return']).cumprod() * CAPITAL
```
Simple returns compounded correctly. Log returns add no meaningful accuracy at a 60-day horizon.

**Annualised Return Approximation**
```python
bars_per_year = 252 * 78  # 252 sessions × 78 five-minute bars
annualised_return = df['Strategy_Return'].mean() * bars_per_year
```
Arithmetic scaling — reasonable at this stage. Geometric annualisation is more rigorous but immaterial at this sample size.

---

### Limitations Identified

| Gap | Impact |
|-----|--------|
| No transaction costs or slippage | Overstates net performance |
| No stop-loss or take-profit | Unrealistic trade management |
| Bar-level P&L only | Win rate and avg R not calculable |
| No Sharpe ratio or max drawdown | Incomplete risk picture |
| 60-day sample window | Insufficient for statistical significance |

---

### Critical Reflection

EMA crossovers are among the most studied and most overfitted strategies in retail trading. On 5-minute NQ data, the system performs well in directional sessions and deteriorates sharply in choppy, low-volatility regimes. The annualised return figure carries no statistical weight at this sample size.

**The goal of this script was not to find an edge.** It was to build and fully understand the core backtest pipeline: data → indicators → signals → returns → equity curve. That objective was met.

---

### Open Questions

- [ ] How do I correctly deduct costs at the trade level rather than the bar level?
- [ ] How do I reconstruct trade-by-trade P&L from a vectorised signal series?
- [ ] What is the difference between vectorised and event-driven backtesting frameworks?
- [ ] How does `yfinance` data quality compare to exchange-grade tick data?
- [ ] What sample size is needed for statistically significant backtest conclusions?

---

### Next Build Targets

- [ ] Trade-level metrics: win rate, avg win, avg loss, profit factor
- [ ] Risk metrics: Sharpe ratio, max drawdown, Calmar ratio
- [ ] Proper cost model: deduct on entry and exit bars only
- [ ] Regime filter: ATR or volume condition to suppress signals in choppy markets
- [ ] Systematic parameter sweep across EMA pairs: 5/13 · 9/21 · 20/50
