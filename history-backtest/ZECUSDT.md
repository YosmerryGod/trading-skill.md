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


## Run — 7 hari (2026-11-11)

- Status: RUN_COMPLETE_GROSS
- Evaluation period UTC: 2026-11-03T21:43:00.000Z to 2026-11-10T21:43:00.000Z (end exclusive)
- Exchange/contract: Binance USDⓈ-M perpetual futures, ZECUSDT
- PSAR: custom implementation, step 0.01, max AF 0.10; not verified against TradingView. Initial trend seeded from first two closes.
- 1H filter: latest fully closed 1H candle at signal time; 5M entry only on newly crossed body beyond PSAR.
- Entry: first 1M open after signal close; notional 100 USDT.
- Volatility: 288 prior completed 5M candles; formula per backtest-skill.md; buckets kept exact, gaps unbucketed.
- Exit: sequential 1M OHLC; simultaneous TP/SL => SL-first; gap-through fill uses candle open.
- Position control: one open position at a time; new signal skipped while a position is open. Three consecutive losses trigger 24h cooldown.
- Costs: GROSS — fees, slippage and funding excluded.

### Data validation

| Timeframe | Candles | Gaps | Duplicates |
|---|---:|---:|---:|
| 1M | 10080 | 0 | 0 |
| 5M | 2304 | 0 | 0 |
| 1H | 288 | 0 | 0 |

### Signal and volatility distribution

- Valid signals: 41; rejected by 1H trend filter: 49.
- V1: 29 signals.
- V2: 3 signals.
- V3: 0 signals.
- V4: 0 signals.
- UNCATEGORIZED_GAP: 9 signals.

### ALL_MARKETS summary

| TP/SL | Trades | TP | SL | Win rate | Gross PnL USDT | PF | Expectancy USDT | Max DD |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| A | 16 | 8 | 8 | 50.00% | 8.00 | 1.50 | 0.50 | 7.62% |
| B | 8 | 3 | 5 | 37.50% | -0.00 | 1.00 | -0.00 | 15.00% |
| C | 7 | 3 | 4 | 42.86% | 1.00 | 1.05 | 0.14 | 20.00% |
| D | 4 | 3 | 1 | 75.00% | 23.00 | 4.29 | 5.75 | 6.36% |

### Volatility mode comparison — all TP/SL configurations

| Config | Mode | Trades | TP | SL | Win rate | Gross PnL | PF | Max DD | Skipped vol | Skipped cooldown | Skipped open-position |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| A | ALL_MARKETS | 16 | 8 | 8 | 50.00% | 8.00 | 1.50 | 7.62% | 0 | 5 | 20 |
| A | ONLY_V1 | 12 | 7 | 5 | 58.33% | 11.00 | 2.10 | 5.83% | 12 | 4 | 13 |
| A | ONLY_V2 | 1 | 1 | 0 | 100.00% | 3.00 | — | 0.00% | 38 | 0 | 2 |
| A | ONLY_V3 | 0 | 0 | 0 | —% | 0.00 | — | 0.00% | 41 | 0 | 0 |
| A | ONLY_V4 | 0 | 0 | 0 | —% | 0.00 | — | 0.00% | 41 | 0 | 0 |
| A | EXCLUDE_V1 | 6 | 2 | 4 | 33.33% | -2.00 | 0.75 | 4.00% | 29 | 0 | 6 |
| A | EXCLUDE_V2 | 15 | 7 | 8 | 46.67% | 5.00 | 1.31 | 7.84% | 3 | 5 | 18 |
| A | EXCLUDE_V3 | 16 | 8 | 8 | 50.00% | 8.00 | 1.50 | 7.62% | 0 | 5 | 20 |
| A | EXCLUDE_V4 | 16 | 8 | 8 | 50.00% | 8.00 | 1.50 | 7.62% | 0 | 5 | 20 |
| B | ALL_MARKETS | 8 | 3 | 5 | 37.50% | -0.00 | 1.00 | 15.00% | 0 | 16 | 17 |
| B | ONLY_V1 | 9 | 6 | 3 | 66.67% | 21.00 | 3.33 | 5.71% | 12 | 0 | 20 |
| B | ONLY_V2 | 1 | 0 | 1 | 0.00% | -3.00 | 0.00 | 3.00% | 38 | 0 | 2 |
| B | ONLY_V3 | 0 | 0 | 0 | —% | 0.00 | — | 0.00% | 41 | 0 | 0 |
| B | ONLY_V4 | 0 | 0 | 0 | —% | 0.00 | — | 0.00% | 41 | 0 | 0 |
| B | EXCLUDE_V1 | 5 | 0 | 5 | 0.00% | -15.00 | 0.00 | 15.00% | 29 | 6 | 1 |
| B | EXCLUDE_V2 | 8 | 3 | 5 | 37.50% | -0.00 | 1.00 | 15.00% | 3 | 13 | 17 |
| B | EXCLUDE_V3 | 8 | 3 | 5 | 37.50% | -0.00 | 1.00 | 15.00% | 0 | 16 | 17 |
| B | EXCLUDE_V4 | 8 | 3 | 5 | 37.50% | -0.00 | 1.00 | 15.00% | 0 | 16 | 17 |
| C | ALL_MARKETS | 7 | 3 | 4 | 42.86% | 1.00 | 1.05 | 20.00% | 0 | 7 | 27 |
| C | ONLY_V1 | 7 | 5 | 2 | 71.43% | 25.00 | 3.50 | 4.67% | 12 | 0 | 22 |
| C | ONLY_V2 | 1 | 0 | 1 | 0.00% | -5.00 | 0.00 | 5.00% | 38 | 0 | 2 |
| C | ONLY_V3 | 0 | 0 | 0 | —% | 0.00 | — | 0.00% | 41 | 0 | 0 |
| C | ONLY_V4 | 0 | 0 | 0 | —% | 0.00 | — | 0.00% | 41 | 0 | 0 |
| C | EXCLUDE_V1 | 3 | 0 | 3 | 0.00% | -15.00 | 0.00 | 15.00% | 29 | 2 | 7 |
| C | EXCLUDE_V2 | 7 | 3 | 4 | 42.86% | 1.00 | 1.05 | 20.00% | 3 | 7 | 24 |
| C | EXCLUDE_V3 | 7 | 3 | 4 | 42.86% | 1.00 | 1.05 | 20.00% | 0 | 7 | 27 |
| C | EXCLUDE_V4 | 7 | 3 | 4 | 42.86% | 1.00 | 1.05 | 20.00% | 0 | 7 | 27 |
| D | ALL_MARKETS | 4 | 3 | 1 | 75.00% | 23.00 | 4.29 | 6.36% | 0 | 0 | 37 |
| D | ONLY_V1 | 2 | 2 | 0 | 100.00% | 20.00 | — | 0.00% | 12 | 0 | 27 |
| D | ONLY_V2 | 1 | 0 | 1 | 0.00% | -7.00 | 0.00 | 7.00% | 38 | 0 | 2 |
| D | ONLY_V3 | 0 | 0 | 0 | —% | 0.00 | — | 0.00% | 41 | 0 | 0 |
| D | ONLY_V4 | 0 | 0 | 0 | —% | 0.00 | — | 0.00% | 41 | 0 | 0 |
| D | EXCLUDE_V1 | 2 | 1 | 1 | 50.00% | 3.00 | 1.43 | 6.36% | 29 | 0 | 10 |
| D | EXCLUDE_V2 | 4 | 3 | 1 | 75.00% | 23.00 | 4.29 | 6.36% | 3 | 0 | 34 |
| D | EXCLUDE_V3 | 4 | 3 | 1 | 75.00% | 23.00 | 4.29 | 6.36% | 0 | 0 | 37 |
| D | EXCLUDE_V4 | 4 | 3 | 1 | 75.00% | 23.00 | 4.29 | 6.36% | 0 | 0 | 37 |

### Daily breakdown — exit day UTC; zero-trade days included

| Date | Config | Mode | Signals | Closed trades | TP | SL | Win rate | Gross PnL USDT |
|---|---|---|---:|---:|---:|---:|---:|---:|
| 2026-11-03 | A | ALL_MARKETS | 6 | 3 | 1 | 2 | 33.33% | -1.00 |
| 2026-11-04 | A | ALL_MARKETS | 8 | 4 | 2 | 2 | 50.00% | 2.00 |
| 2026-11-05 | A | ALL_MARKETS | 6 | 1 | 0 | 1 | 0.00% | -2.00 |
| 2026-11-06 | A | ALL_MARKETS | 6 | 2 | 1 | 1 | 50.00% | 1.00 |
| 2026-11-07 | A | ALL_MARKETS | 4 | 2 | 1 | 1 | 50.00% | 1.00 |
| 2026-11-08 | A | ALL_MARKETS | 7 | 2 | 2 | 0 | 100.00% | 6.00 |
| 2026-11-09 | A | ALL_MARKETS | 4 | 2 | 1 | 1 | 50.00% | 1.00 |
| 2026-11-03 | A | ONLY_V1 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-04 | A | ONLY_V1 | 5 | 3 | 1 | 2 | 33.33% | -1.00 |
| 2026-11-05 | A | ONLY_V1 | 5 | 1 | 0 | 1 | 0.00% | -2.00 |
| 2026-11-06 | A | ONLY_V1 | 4 | 2 | 2 | 0 | 100.00% | 6.00 |
| 2026-11-07 | A | ONLY_V1 | 4 | 2 | 1 | 1 | 50.00% | 1.00 |
| 2026-11-08 | A | ONLY_V1 | 7 | 2 | 2 | 0 | 100.00% | 6.00 |
| 2026-11-09 | A | ONLY_V1 | 4 | 2 | 1 | 1 | 50.00% | 1.00 |
| 2026-11-03 | A | ONLY_V2 | 1 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-04 | A | ONLY_V2 | 2 | 1 | 1 | 0 | 100.00% | 3.00 |
| 2026-11-05 | A | ONLY_V2 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-06 | A | ONLY_V2 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-07 | A | ONLY_V2 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | A | ONLY_V2 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-09 | A | ONLY_V2 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-03 | A | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-04 | A | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-05 | A | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-06 | A | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-07 | A | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | A | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-09 | A | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-03 | A | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-04 | A | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-05 | A | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-06 | A | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-07 | A | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | A | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-09 | A | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-03 | A | EXCLUDE_V1 | 6 | 3 | 1 | 2 | 33.33% | -1.00 |
| 2026-11-04 | A | EXCLUDE_V1 | 3 | 1 | 1 | 0 | 100.00% | 3.00 |
| 2026-11-05 | A | EXCLUDE_V1 | 1 | 1 | 0 | 1 | 0.00% | -2.00 |
| 2026-11-06 | A | EXCLUDE_V1 | 2 | 1 | 0 | 1 | 0.00% | -2.00 |
| 2026-11-07 | A | EXCLUDE_V1 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | A | EXCLUDE_V1 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-09 | A | EXCLUDE_V1 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-03 | A | EXCLUDE_V2 | 5 | 3 | 1 | 2 | 33.33% | -1.00 |
| 2026-11-04 | A | EXCLUDE_V2 | 6 | 3 | 1 | 2 | 33.33% | -1.00 |
| 2026-11-05 | A | EXCLUDE_V2 | 6 | 1 | 0 | 1 | 0.00% | -2.00 |
| 2026-11-06 | A | EXCLUDE_V2 | 6 | 2 | 1 | 1 | 50.00% | 1.00 |
| 2026-11-07 | A | EXCLUDE_V2 | 4 | 2 | 1 | 1 | 50.00% | 1.00 |
| 2026-11-08 | A | EXCLUDE_V2 | 7 | 2 | 2 | 0 | 100.00% | 6.00 |
| 2026-11-09 | A | EXCLUDE_V2 | 4 | 2 | 1 | 1 | 50.00% | 1.00 |
| 2026-11-03 | A | EXCLUDE_V3 | 6 | 3 | 1 | 2 | 33.33% | -1.00 |
| 2026-11-04 | A | EXCLUDE_V3 | 8 | 4 | 2 | 2 | 50.00% | 2.00 |
| 2026-11-05 | A | EXCLUDE_V3 | 6 | 1 | 0 | 1 | 0.00% | -2.00 |
| 2026-11-06 | A | EXCLUDE_V3 | 6 | 2 | 1 | 1 | 50.00% | 1.00 |
| 2026-11-07 | A | EXCLUDE_V3 | 4 | 2 | 1 | 1 | 50.00% | 1.00 |
| 2026-11-08 | A | EXCLUDE_V3 | 7 | 2 | 2 | 0 | 100.00% | 6.00 |
| 2026-11-09 | A | EXCLUDE_V3 | 4 | 2 | 1 | 1 | 50.00% | 1.00 |
| 2026-11-03 | A | EXCLUDE_V4 | 6 | 3 | 1 | 2 | 33.33% | -1.00 |
| 2026-11-04 | A | EXCLUDE_V4 | 8 | 4 | 2 | 2 | 50.00% | 2.00 |
| 2026-11-05 | A | EXCLUDE_V4 | 6 | 1 | 0 | 1 | 0.00% | -2.00 |
| 2026-11-06 | A | EXCLUDE_V4 | 6 | 2 | 1 | 1 | 50.00% | 1.00 |
| 2026-11-07 | A | EXCLUDE_V4 | 4 | 2 | 1 | 1 | 50.00% | 1.00 |
| 2026-11-08 | A | EXCLUDE_V4 | 7 | 2 | 2 | 0 | 100.00% | 6.00 |
| 2026-11-09 | A | EXCLUDE_V4 | 4 | 2 | 1 | 1 | 50.00% | 1.00 |
| 2026-11-03 | B | ALL_MARKETS | 6 | 2 | 0 | 2 | 0.00% | -6.00 |
| 2026-11-04 | B | ALL_MARKETS | 8 | 1 | 0 | 1 | 0.00% | -3.00 |
| 2026-11-05 | B | ALL_MARKETS | 6 | 1 | 0 | 1 | 0.00% | -3.00 |
| 2026-11-06 | B | ALL_MARKETS | 6 | 1 | 0 | 1 | 0.00% | -3.00 |
| 2026-11-07 | B | ALL_MARKETS | 4 | 1 | 1 | 0 | 100.00% | 5.00 |
| 2026-11-08 | B | ALL_MARKETS | 7 | 1 | 1 | 0 | 100.00% | 5.00 |
| 2026-11-09 | B | ALL_MARKETS | 4 | 1 | 1 | 0 | 100.00% | 5.00 |
| 2026-11-03 | B | ONLY_V1 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-04 | B | ONLY_V1 | 5 | 1 | 1 | 0 | 100.00% | 5.00 |
| 2026-11-05 | B | ONLY_V1 | 5 | 4 | 1 | 3 | 25.00% | -4.00 |
| 2026-11-06 | B | ONLY_V1 | 4 | 1 | 1 | 0 | 100.00% | 5.00 |
| 2026-11-07 | B | ONLY_V1 | 4 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | B | ONLY_V1 | 7 | 2 | 2 | 0 | 100.00% | 10.00 |
| 2026-11-09 | B | ONLY_V1 | 4 | 1 | 1 | 0 | 100.00% | 5.00 |
| 2026-11-03 | B | ONLY_V2 | 1 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-04 | B | ONLY_V2 | 2 | 1 | 0 | 1 | 0.00% | -3.00 |
| 2026-11-05 | B | ONLY_V2 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-06 | B | ONLY_V2 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-07 | B | ONLY_V2 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | B | ONLY_V2 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-09 | B | ONLY_V2 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-03 | B | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-04 | B | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-05 | B | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-06 | B | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-07 | B | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | B | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-09 | B | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-03 | B | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-04 | B | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-05 | B | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-06 | B | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-07 | B | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | B | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-09 | B | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-03 | B | EXCLUDE_V1 | 6 | 2 | 0 | 2 | 0.00% | -6.00 |
| 2026-11-04 | B | EXCLUDE_V1 | 3 | 1 | 0 | 1 | 0.00% | -3.00 |
| 2026-11-05 | B | EXCLUDE_V1 | 1 | 1 | 0 | 1 | 0.00% | -3.00 |
| 2026-11-06 | B | EXCLUDE_V1 | 2 | 1 | 0 | 1 | 0.00% | -3.00 |
| 2026-11-07 | B | EXCLUDE_V1 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | B | EXCLUDE_V1 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-09 | B | EXCLUDE_V1 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-03 | B | EXCLUDE_V2 | 5 | 2 | 0 | 2 | 0.00% | -6.00 |
| 2026-11-04 | B | EXCLUDE_V2 | 6 | 1 | 0 | 1 | 0.00% | -3.00 |
| 2026-11-05 | B | EXCLUDE_V2 | 6 | 1 | 0 | 1 | 0.00% | -3.00 |
| 2026-11-06 | B | EXCLUDE_V2 | 6 | 1 | 0 | 1 | 0.00% | -3.00 |
| 2026-11-07 | B | EXCLUDE_V2 | 4 | 1 | 1 | 0 | 100.00% | 5.00 |
| 2026-11-08 | B | EXCLUDE_V2 | 7 | 1 | 1 | 0 | 100.00% | 5.00 |
| 2026-11-09 | B | EXCLUDE_V2 | 4 | 1 | 1 | 0 | 100.00% | 5.00 |
| 2026-11-03 | B | EXCLUDE_V3 | 6 | 2 | 0 | 2 | 0.00% | -6.00 |
| 2026-11-04 | B | EXCLUDE_V3 | 8 | 1 | 0 | 1 | 0.00% | -3.00 |
| 2026-11-05 | B | EXCLUDE_V3 | 6 | 1 | 0 | 1 | 0.00% | -3.00 |
| 2026-11-06 | B | EXCLUDE_V3 | 6 | 1 | 0 | 1 | 0.00% | -3.00 |
| 2026-11-07 | B | EXCLUDE_V3 | 4 | 1 | 1 | 0 | 100.00% | 5.00 |
| 2026-11-08 | B | EXCLUDE_V3 | 7 | 1 | 1 | 0 | 100.00% | 5.00 |
| 2026-11-09 | B | EXCLUDE_V3 | 4 | 1 | 1 | 0 | 100.00% | 5.00 |
| 2026-11-03 | B | EXCLUDE_V4 | 6 | 2 | 0 | 2 | 0.00% | -6.00 |
| 2026-11-04 | B | EXCLUDE_V4 | 8 | 1 | 0 | 1 | 0.00% | -3.00 |
| 2026-11-05 | B | EXCLUDE_V4 | 6 | 1 | 0 | 1 | 0.00% | -3.00 |
| 2026-11-06 | B | EXCLUDE_V4 | 6 | 1 | 0 | 1 | 0.00% | -3.00 |
| 2026-11-07 | B | EXCLUDE_V4 | 4 | 1 | 1 | 0 | 100.00% | 5.00 |
| 2026-11-08 | B | EXCLUDE_V4 | 7 | 1 | 1 | 0 | 100.00% | 5.00 |
| 2026-11-09 | B | EXCLUDE_V4 | 4 | 1 | 1 | 0 | 100.00% | 5.00 |
| 2026-11-03 | C | ALL_MARKETS | 6 | 1 | 0 | 1 | 0.00% | -5.00 |
| 2026-11-04 | C | ALL_MARKETS | 8 | 1 | 0 | 1 | 0.00% | -5.00 |
| 2026-11-05 | C | ALL_MARKETS | 6 | 1 | 0 | 1 | 0.00% | -5.00 |
| 2026-11-06 | C | ALL_MARKETS | 6 | 1 | 0 | 1 | 0.00% | -5.00 |
| 2026-11-07 | C | ALL_MARKETS | 4 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | C | ALL_MARKETS | 7 | 1 | 1 | 0 | 100.00% | 7.00 |
| 2026-11-09 | C | ALL_MARKETS | 4 | 2 | 2 | 0 | 100.00% | 14.00 |
| 2026-11-03 | C | ONLY_V1 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-04 | C | ONLY_V1 | 5 | 1 | 1 | 0 | 100.00% | 7.00 |
| 2026-11-05 | C | ONLY_V1 | 5 | 2 | 1 | 1 | 50.00% | 2.00 |
| 2026-11-06 | C | ONLY_V1 | 4 | 1 | 0 | 1 | 0.00% | -5.00 |
| 2026-11-07 | C | ONLY_V1 | 4 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | C | ONLY_V1 | 7 | 1 | 1 | 0 | 100.00% | 7.00 |
| 2026-11-09 | C | ONLY_V1 | 4 | 2 | 2 | 0 | 100.00% | 14.00 |
| 2026-11-03 | C | ONLY_V2 | 1 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-04 | C | ONLY_V2 | 2 | 1 | 0 | 1 | 0.00% | -5.00 |
| 2026-11-05 | C | ONLY_V2 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-06 | C | ONLY_V2 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-07 | C | ONLY_V2 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | C | ONLY_V2 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-09 | C | ONLY_V2 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-03 | C | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-04 | C | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-05 | C | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-06 | C | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-07 | C | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | C | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-09 | C | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-03 | C | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-04 | C | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-05 | C | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-06 | C | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-07 | C | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | C | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-09 | C | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-03 | C | EXCLUDE_V1 | 6 | 1 | 0 | 1 | 0.00% | -5.00 |
| 2026-11-04 | C | EXCLUDE_V1 | 3 | 1 | 0 | 1 | 0.00% | -5.00 |
| 2026-11-05 | C | EXCLUDE_V1 | 1 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-06 | C | EXCLUDE_V1 | 2 | 1 | 0 | 1 | 0.00% | -5.00 |
| 2026-11-07 | C | EXCLUDE_V1 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | C | EXCLUDE_V1 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-09 | C | EXCLUDE_V1 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-03 | C | EXCLUDE_V2 | 5 | 1 | 0 | 1 | 0.00% | -5.00 |
| 2026-11-04 | C | EXCLUDE_V2 | 6 | 1 | 0 | 1 | 0.00% | -5.00 |
| 2026-11-05 | C | EXCLUDE_V2 | 6 | 1 | 0 | 1 | 0.00% | -5.00 |
| 2026-11-06 | C | EXCLUDE_V2 | 6 | 1 | 0 | 1 | 0.00% | -5.00 |
| 2026-11-07 | C | EXCLUDE_V2 | 4 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | C | EXCLUDE_V2 | 7 | 1 | 1 | 0 | 100.00% | 7.00 |
| 2026-11-09 | C | EXCLUDE_V2 | 4 | 2 | 2 | 0 | 100.00% | 14.00 |
| 2026-11-03 | C | EXCLUDE_V3 | 6 | 1 | 0 | 1 | 0.00% | -5.00 |
| 2026-11-04 | C | EXCLUDE_V3 | 8 | 1 | 0 | 1 | 0.00% | -5.00 |
| 2026-11-05 | C | EXCLUDE_V3 | 6 | 1 | 0 | 1 | 0.00% | -5.00 |
| 2026-11-06 | C | EXCLUDE_V3 | 6 | 1 | 0 | 1 | 0.00% | -5.00 |
| 2026-11-07 | C | EXCLUDE_V3 | 4 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | C | EXCLUDE_V3 | 7 | 1 | 1 | 0 | 100.00% | 7.00 |
| 2026-11-09 | C | EXCLUDE_V3 | 4 | 2 | 2 | 0 | 100.00% | 14.00 |
| 2026-11-03 | C | EXCLUDE_V4 | 6 | 1 | 0 | 1 | 0.00% | -5.00 |
| 2026-11-04 | C | EXCLUDE_V4 | 8 | 1 | 0 | 1 | 0.00% | -5.00 |
| 2026-11-05 | C | EXCLUDE_V4 | 6 | 1 | 0 | 1 | 0.00% | -5.00 |
| 2026-11-06 | C | EXCLUDE_V4 | 6 | 1 | 0 | 1 | 0.00% | -5.00 |
| 2026-11-07 | C | EXCLUDE_V4 | 4 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | C | EXCLUDE_V4 | 7 | 1 | 1 | 0 | 100.00% | 7.00 |
| 2026-11-09 | C | EXCLUDE_V4 | 4 | 2 | 2 | 0 | 100.00% | 14.00 |
| 2026-11-03 | D | ALL_MARKETS | 6 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-04 | D | ALL_MARKETS | 8 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-05 | D | ALL_MARKETS | 6 | 1 | 1 | 0 | 100.00% | 10.00 |
| 2026-11-06 | D | ALL_MARKETS | 6 | 1 | 0 | 1 | 0.00% | -7.00 |
| 2026-11-07 | D | ALL_MARKETS | 4 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | D | ALL_MARKETS | 7 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-09 | D | ALL_MARKETS | 4 | 2 | 2 | 0 | 100.00% | 20.00 |
| 2026-11-03 | D | ONLY_V1 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-04 | D | ONLY_V1 | 5 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-05 | D | ONLY_V1 | 5 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-06 | D | ONLY_V1 | 4 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-07 | D | ONLY_V1 | 4 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | D | ONLY_V1 | 7 | 1 | 1 | 0 | 100.00% | 10.00 |
| 2026-11-09 | D | ONLY_V1 | 4 | 1 | 1 | 0 | 100.00% | 10.00 |
| 2026-11-03 | D | ONLY_V2 | 1 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-04 | D | ONLY_V2 | 2 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-05 | D | ONLY_V2 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-06 | D | ONLY_V2 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-07 | D | ONLY_V2 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | D | ONLY_V2 | 0 | 1 | 0 | 1 | 0.00% | -7.00 |
| 2026-11-09 | D | ONLY_V2 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-03 | D | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-04 | D | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-05 | D | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-06 | D | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-07 | D | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | D | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-09 | D | ONLY_V3 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-03 | D | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-04 | D | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-05 | D | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-06 | D | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-07 | D | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | D | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-09 | D | ONLY_V4 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-03 | D | EXCLUDE_V1 | 6 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-04 | D | EXCLUDE_V1 | 3 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-05 | D | EXCLUDE_V1 | 1 | 1 | 1 | 0 | 100.00% | 10.00 |
| 2026-11-06 | D | EXCLUDE_V1 | 2 | 1 | 0 | 1 | 0.00% | -7.00 |
| 2026-11-07 | D | EXCLUDE_V1 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | D | EXCLUDE_V1 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-09 | D | EXCLUDE_V1 | 0 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-03 | D | EXCLUDE_V2 | 5 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-04 | D | EXCLUDE_V2 | 6 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-05 | D | EXCLUDE_V2 | 6 | 1 | 1 | 0 | 100.00% | 10.00 |
| 2026-11-06 | D | EXCLUDE_V2 | 6 | 1 | 0 | 1 | 0.00% | -7.00 |
| 2026-11-07 | D | EXCLUDE_V2 | 4 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | D | EXCLUDE_V2 | 7 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-09 | D | EXCLUDE_V2 | 4 | 2 | 2 | 0 | 100.00% | 20.00 |
| 2026-11-03 | D | EXCLUDE_V3 | 6 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-04 | D | EXCLUDE_V3 | 8 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-05 | D | EXCLUDE_V3 | 6 | 1 | 1 | 0 | 100.00% | 10.00 |
| 2026-11-06 | D | EXCLUDE_V3 | 6 | 1 | 0 | 1 | 0.00% | -7.00 |
| 2026-11-07 | D | EXCLUDE_V3 | 4 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | D | EXCLUDE_V3 | 7 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-09 | D | EXCLUDE_V3 | 4 | 2 | 2 | 0 | 100.00% | 20.00 |
| 2026-11-03 | D | EXCLUDE_V4 | 6 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-04 | D | EXCLUDE_V4 | 8 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-05 | D | EXCLUDE_V4 | 6 | 1 | 1 | 0 | 100.00% | 10.00 |
| 2026-11-06 | D | EXCLUDE_V4 | 6 | 1 | 0 | 1 | 0.00% | -7.00 |
| 2026-11-07 | D | EXCLUDE_V4 | 4 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-08 | D | EXCLUDE_V4 | 7 | 0 | 0 | 0 | —% | 0.00 |
| 2026-11-09 | D | EXCLUDE_V4 | 4 | 2 | 2 | 0 | 100.00% | 20.00 |

### Limitations and conclusion

- PSAR is custom and unverified against TradingView; results are exploratory.
- Fee, slippage and funding excluded; all PnL is gross. Notional is 100 USDT per trade.
- Only 7-day period was run here; no claim about 30 days.
- V3 and V4 had no valid signals in this sample; their isolated-mode results are zero-trade results, not evidence of poor performance.
- Results do not establish profitability or robustness; do not use as live-entry advice.
- Decision: INCONCLUSIVE pending longer and out-of-sample validation.


## Correction note — 2026-11-11

The earlier one-day run recorded above did not correctly enforce the one-open-position-at-a-time rule across overlapping signals. Its trade counts and PnL are therefore RETRACTED and must not be used for strategy evaluation. The seven-day run appended below was re-simulated with the single-position constraint enforced. The one-day period must be rerun under the corrected engine before it can be considered valid.
