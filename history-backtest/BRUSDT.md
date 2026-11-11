# BRUSDT — Backtest History

## Run — 2026-11-11

- Status: `INCOMPLETE_DATA`
- Strategy: Parabolic SAR Follow Trend
- Exchange: Binance USDⓈ-M perpetual
- Contract status: TRADING
- Evaluation timezone: UTC
- Evaluation end: 2026-11-11 00:00:00 UTC
- Evaluated windows: 1 day (2026-11-10), 7 days (2026-11-04 through 2026-11-10)
- 30-day window: NOT RUN — full 1M exit coverage and required warm-up were not retrieved; no result is inferred.
- PSAR: Step 0.01, Maximum 0.10. Custom implementation used for this exploratory run; not independently verified against TradingView or a reference implementation.
- Trend: latest completed 1H candle; entry: completed 5M close-cross.
- Entry fill: signal close adjusted by estimated 0.05% adverse slippage.
- Exit: 1M OHLC sequence; if TP and SL both touched in one 1M candle, SL-first.
- Starting balance: 100 USDT; leverage 3x; Cross Margin; 100% available balance as margin; compounding; one position at a time.
- TP/SL: 3/2, 5/3, 7/5, 10/7.
- Fee assumption: taker 0.05% per side.
- Slippage assumption: 0.05% per side.
- Funding: historical Binance funding rates applied at funding timestamps while positions were open.
- Dynamic ATR: not run (baseline only).

### Table 1 — Baseline summary

PnL and final balance are in USDT. Each period starts independently from 100 USDT. Win rate = TP / (TP + SL); open-at-end positions are excluded from the win-rate denominator and are disclosed separately.

| Period | TP3/SL2 PnL | Trades | Win rate | Final balance | TP5/SL3 PnL | Trades | Win rate | Final balance | TP7/SL5 PnL | Trades | Win rate | Final balance | TP10/SL7 PnL | Trades | Win rate | Final balance |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 1 day | -3.54 | 5 | 40.0% | 96.46 | +7.53 | 4 | 50.0% | 107.53 | +45.26 | 2 | 100% | 145.26 | +29.52 | 1 | 100% | 129.52 |
| 7 days | +13.45 | 16 | 50.0% | 113.45 | +52.17 | 10 | 60.0% | 152.17 | +24.94 | 5 | 60.0% | 124.94 | -38.51 | 2 | 0% | 61.49 |
| 30 days | NOT RUN | — | — | — | NOT RUN | — | — | — | NOT RUN | — | — | — | NOT RUN | — | — | — |

### Table 2 — Daily volatility breakdown (2026-11-10 UTC)

Volatility uses the preceding 288 completed 5M candles. Intervals are lower-bound inclusive and upper-bound exclusive. ALL TRADING is the unfiltered simulation repeated as the comparator for each range. ONLY and EXCLUDE are independent simulations starting from 100 USDT.

| Volatility | Mode | TP3/SL2 PnL | Trades | Win rate | TP5/SL3 PnL | Trades | Win rate | TP7/SL5 PnL | Trades | Win rate | TP10/SL7 PnL | Trades | Win rate |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 15–20% | ALL TRADING | -3.54 | 5 | 40.0% | +7.53 | 4 | 50.0% | +45.26 | 2 | 100% | +29.52 | 1 | 100% |
| 15–20% | ONLY | +1.55 | 2 | 50.0% | +14.54 | 1 | 100% | +20.53 | 1 | 100% | +29.52 | 1 | 100% |
| 15–20% | EXCLUDE | -5.01 | 3 | 33.3% | -6.11 | 3 | 33.3% | +20.51 | 1 | 100% | 0.00 | 0 | — |
| 20–25% | ALL TRADING | -3.54 | 5 | 40.0% | +7.53 | 4 | 50.0% | +45.26 | 2 | 100% | +29.52 | 1 | 100% |
| 20–25% | ONLY | -5.01 | 3 | 33.3% | -6.11 | 3 | 33.3% | +20.51 | 1 | 100% | 0.00 | 0 | — |
| 20–25% | EXCLUDE | +1.55 | 2 | 50.0% | +14.54 | 1 | 100% | +20.53 | 1 | 100% | +29.52 | 1 | 100% |

### Data quality and limitations

- 1-day candles: 1,440 × 1M, 864 × 5M (including warm-up), 250 × 1H. No timestamp gaps detected in the fetched ranges.
- 7-day candles: 10,080 × 1M, 2,592 × 5M (including warm-up), 418 × 1H. No timestamp gaps detected in the fetched ranges.
- Funding records: 6 for the 1-day window; 42 for the 7-day window.
- Open-at-end positions: one in each ALL baseline configuration for both periods; their unrealized PnL is not included in realized PnL.
- Liquidation: exact historical leverage-bracket tiers were not available in the retrieved data, so liquidation was not independently simulated. Do not treat this as a verified zero-liquidation result.
- Exchange info returned current symbol filters and base maintenance-margin fields, not the full historical leverage-bracket schedule.
- Fee and slippage are modeled assumptions, not the user's account-specific fee or actual historical fill.
- PSAR initialization/implementation has not been cross-validated; results are exploratory and must be reproduced with a validated PSAR implementation before being treated as final.
- No raw trade/skip CSV was exported in this run.
- 30-day result remains untested; do not infer it from the 1-day or 7-day results.
- The short sample and small trade counts do not establish profitability or robustness.

