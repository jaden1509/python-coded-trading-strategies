## RSI Mean Reversion Backtest

**Date:** 2026-05-26
**Script:** `rsi_mean_reversion.py`
**Asset:** NQ=F (Nasdaq-100 Futures)
**Timeframe:** 5-minute bars
**Lookback:** 60 days
**Stack:** `yfinance` · `pandas` · `numpy` · `matplotlib`

---

### Strategy Logic

- RSI crosses below 30 (oversold) → Long (signal = 1)
- RSI crosses above 70 (overbought) → Flat (signal = 0)
- Long-only · no short side · no stop-loss

---

### Key Concepts Learned

**RSI calculation from scratch**
```python
delta    = df['Close'].diff()
gain     = delta.where(delta > 0, 0)
loss     = -delta.where(delta < 0, 0)
avg_gain = gain.rolling(RSI_LEN).mean()
avg_loss = loss.rolling(RSI_LEN).mean()
rs       = avg_gain / avg_loss
df['RSI'] = 100 - (100 / (1 + rs))
```
- Built RSI manually rather than calling a library — forces understanding of what each step computes
- `delta.where(condition, 0)` isolates gains and losses: positive deltas for gains, negated negatives for losses
- RS is the ratio of average gain to average loss; RSI normalises this to a 0–100 scale
- Key parameters: `RSI_LEN` controls lookback (standard = 14), `RSI_OB` / `RSI_OS` set overbought and oversold thresholds

**Sparse signal construction with `ffill()`**
```python
df['raw_signal'] = np.nan
df.loc[df['RSI'] < RSI_OS, 'raw_signal'] = 1
df.loc[df['RSI'] > RSI_OB, 'raw_signal'] = 0
df['Signal'] = df['raw_signal'].ffill().fillna(0)
```
- RSI thresholds only fire occasionally — most bars have no signal event, unlike EMA crossover
- Setting non-event bars to `NaN` then using `ffill()` carries the last known signal forward until the next crossing
- `.fillna(0)` handles bars before the first signal fires — defaults to flat

**Geometric annualisation**
```python
days_traded = (df.index[-1] - df.index[0]).days
ann_return  = ((1 + total_return/100) ** (365/days_traded) - 1) * 100
```
- Replaced the arithmetic scaling from Entry 001 with the proper compounding formula
- Scales total return over exact calendar days — more accurate than multiplying mean bar return by bars per year

**Revisited from Entry 001**
- Had to go back and revisit `.diff()`, `.shift()`, `(1 + r).cumprod()` — confirms the same pipeline structure repeats across strategies
- Next step: compile a dedicated `.methods()` and `(arguments)` reference sheet — a running list of every pandas/numpy method used, with parameters, for memorisation and future lookup

---

### Limitations Identified

| Gap | Impact |
|-----|--------|
| Simple rolling mean for RSI | Real RSI uses Wilder's smoothing — values will differ from TradingView |
| No transaction costs or slippage | Overstates net performance |
| No stop-loss or take-profit | Unrealistic trade management |
| Bar-level P&L only | Win rate and avg R not calculable |
| 60-day sample window | Insufficient for statistical significance |

---

### Notes

NQ is a trending instrument — during strong momentum sessions, an oversold RSI can keep pushing lower for many bars before reversing, causing extended drawdowns with no exit mechanism. Without a stop-loss the strategy has unlimited downside on any given trade.

**The meaningful step forward in this script** was building RSI from raw price data and learning how to construct a sparse signal using `ffill()` — a pattern that applies to any threshold-based strategy.

---

### Open Questions

- [ ] None for now

---

### Next Build Targets

- [ ] Compile a `.methods()` and `(arguments)` reference sheet for all pandas/numpy methods used so far
- [ ] Trade-level metrics: win rate, avg win, avg loss, profit factor
- [ ] Risk metrics: Sharpe ratio, max drawdown, Sortino ratio
- [ ] Compare RSI mean reversion vs EMA crossover on the same data window
- [ ] Test whether a time filter (e.g. avoid first 30 minutes) improves performance
