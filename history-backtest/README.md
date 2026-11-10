# Backtest History

This folder stores the completed historical backtest reports, organized by market symbol.

## Mandatory file naming

After a backtest has actually been executed, save its report in a symbol-specific Markdown file:

- `history-backtest/BTCUSDT.md`
- `history-backtest/ETHUSDT.md`
- `history-backtest/BRUSDT.md`

Use the exact uppercase Binance Futures symbol in the filename. Create the file when the first completed or attempted backtest report for that symbol is ready to be recorded.

## Report rules

1. Read the repository's [backtest-skill.md](../backtest-skill.md) and follow its required methodology.
2. Keep all backtest runs for the same symbol in that symbol's file.
3. Append each new run as a new dated section. **Do not overwrite or delete previous runs.**
4. Record the strategy version, TP/SL configuration, volatility mode, evaluation period, data source, and UTC date range for every run.
5. Include the actual results, daily breakdowns, trade-log references, data-quality issues, assumptions, and final status required by the backtest specification.
6. If the backtest was not executed, do not create a fake result. If a report is needed to document an attempted but incomplete run, clearly mark it `INCOMPLETE_DATA` or the applicable status.
7. Never invent trade counts, win rates, PnL, or any other metrics. Distinguish measured results from assumptions.
8. If raw CSV/JSON logs are available, save them alongside the symbol report in a clearly named subfolder or file and link them from the report.

## Suggested report organization

Each symbol file should preserve multiple runs chronologically, newest run appended as a new section:

```markdown
# BTCUSDT — Backtest History

## Run — YYYY-MM-DD

- Status: RUN_COMPLETE_GROSS / RUN_COMPLETE_NET / INCOMPLETE_DATA / INCONCLUSIVE
- Strategy version:
- Evaluation period (UTC):
- TP/SL:
- Volatility mode:
- Data source:
- Results:
- Limitations:
- Trade log:
```

This is only a structural guide. Each run must include all required metrics and details from [backtest-skill.md](../backtest-skill.md); the short outline above is not a substitute for the full report.

## Important

A symbol file is a history ledger, not a place to keep only the best result. Preserve failed, losing, ambiguous, incomplete, and inconclusive runs so future analysis cannot cherry-pick outcomes.
