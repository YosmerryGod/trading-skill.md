# ZECUSDT — Backtest History

## Run — 2026-11-11

- Status: INCONCLUSIVE
- Evaluation period actually simulated: 2026-11-09 21:35:00 UTC to 2026-11-10 21:35:00 UTC (end exclusive)
- Strategy: Parabolic SAR Follow Trend, 1H filter / 5M signal / 1M execution
- Exchange: Binance USDⓈ-M perpetual futures
- PSAR parameters: step 0.01, max AF 0.10
- PSAR implementation: custom implementation used for this exploratory run; not verified against TradingView/reference implementation
- Volatility: prior 288 completed 5M candles; (highest high - lowest low) / first open * 100
- Entry: first 1M open immediately after the signal 5M candle closes
- Notional: 100 USDT per trade
- Costs: GROSS — fees, slippage, and funding excluded
- Exit sequencing: 1M OHLC; simultaneous TP/SL handled conservatively as SL-first
- Data coverage: 1,440 1M candles, 576 5M candles, 148 1H candles; no gaps or duplicates in fetched ranges

### Signal summary

- Valid 5M signals aligned with the 1H filter: 4 (all SHORT)
- Signals rejected by the 1H trend filter: 7
- Volatility bucket: V1 for all 4 valid signals (7.48%–8.61%)
- All signals were in the same V1 bucket; no signals in V2, V3, V4, or uncategorized gaps.

### ALL_MARKETS results

| Configuration | TP/SL | Trades opened | TP | SL | Win rate | Gross PnL | Profit factor | Expectancy | Max drawdown |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A | +3% / -2% | 3 | 0 | 3 | 0% | -6 USDT | 0.00 | -2 USDT | 6% |
| B | +5% / -3% | 3 | 0 | 3 | 0% | -9 USDT | 0.00 | -3 USDT | 9% |
| C | +7% / -5% | 4 | 4 | 0 | 100% | +28 USDT | N/A (no gross loss) | +7 USDT | 0% |
| D | +10% / -7% | 4 | 4 | 0 | 100% | +40 USDT | N/A (no gross loss) | +10 USDT | 0% |

Configuration A and B each had the fourth signal skipped after three consecutive losses triggered the 24-hour cooldown. Configuration C and D did not have a loss streak.

### Volatility experiments

All four valid signals were V1. Therefore ONLY_V1 duplicates the ALL_MARKETS entry candidates for this one-day sample. ONLY_V2/V3/V4 have zero entries. EXCLUDE_V1 has zero entries. Other exclusion modes retain the V1 signals. The simulations for all 9 modes per TP/SL were executed; the same chronology, single-position rule and cooldown rule were applied.

### Limitations

- Only the one-day sample above was simulated. Seven-day and 30-day simulations were NOT completed; do not infer their results from partial candle downloads.
- Four valid signals are far too few for a strategy conclusion.
- The PSAR implementation is custom and has not been verified against TradingView.
- Fees, slippage, and funding are excluded. The results are gross, not net profit.
- C and D are not declared superior based on this sample.
- No live entry recommendation is implied.

### Conclusion

Decision: INCONCLUSIVE. Further data collection and validation are required. This short sample does not establish that the strategy is profitable or robust.
