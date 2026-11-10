# Trading Skill

A focused repository for reproducible cryptocurrency futures strategy research and historical backtesting.

The goal is to define trading rules precisely, test them against historical market data, preserve losing and inconclusive results, and avoid unsupported profitability claims.

## Documentation

| File | Purpose |
|---|---|
| [backtest-skill.md](backtest-skill.md) | Complete Parabolic SAR Follow Trend backtesting protocol |

## Strategy covered

The current backtesting specification defines a Parabolic SAR trend-following strategy:

- **Market:** Binance USDⓈ-M perpetual futures.
- **Trend filter:** 1H Parabolic SAR.
- **Entry signal:** 5M candle close with the full candle body beyond PSAR, aligned with the 1H trend.
- **Entry execution:** First available 1M candle open after the 5M signal candle closes.
- **Exit validation:** Sequential 1M candles for TP/SL checks.
- **Risk control:** One open position at a time; pause new entries for 24 hours after three consecutive losses.
- **TP/SL experiments:** 3%/2%, 5%/3%, 7%/5%, and 10%/7%.
- **Volatility research:** Compare all-market results with isolated and excluded volatility buckets.
- **Evaluation windows:** Completed 1-day, 7-day, and 30-day periods, including daily breakdowns.

Read [backtest-skill.md](backtest-skill.md) before collecting data or running a simulation. Its rules are the source of truth for the current experiment.

## Required research discipline

1. Freeze the strategy rules and parameters before each run.
2. Use only information available at the time of each historical signal.
3. Validate data coverage and timestamp alignment before simulation.
4. Never infer the order of TP and SL when both are touched inside the same 1M candle. Apply the documented conservative assumption and label the ambiguity.
5. Report missing data, skipped signals, open positions, costs, and all losing configurations.
6. Separate gross results from net results after fees, slippage, and funding.
7. Preserve raw trade logs so every reported metric can be audited.
8. Never invent results or claim a backtest was run when only a plan or specification exists.

## Backtest status

Every report must use an explicit status:

- `NOT_RUN`
- `INCOMPLETE_DATA`
- `RUN_COMPLETE_GROSS`
- `RUN_COMPLETE_NET`
- `INCONCLUSIVE`

A positive short-period result is not proof that a strategy is profitable or robust. Longer historical coverage and out-of-sample validation are required.

## Repository structure

```text
.
├── README.md
└── backtest-skill.md
```

## Disclaimer

This repository documents research and simulation methodology. It does not guarantee future performance or provide personalized financial advice. Futures trading involves substantial risk, including the possibility of losing the full amount allocated.
