# 比較股票

Compare 2-5 stocks side by side with key financial metrics.

## Usage

```
/compare_stocks <ticker1> <ticker2> [ticker3] [ticker4] [ticker5]
```

## Examples

```
/compare_stocks AAPL MSFT GOOGL
/compare_stocks 2330 2317 TW
/compare_stocks TSLA NIO
```

## Instructions

When this command is invoked:

1. **Parse arguments:**
   - Extract 2-5 ticker symbols
   - If last argument is a market code (US/TW/CN), apply to all tickers
   - Otherwise auto-detect market per ticker

2. **Cache-first data collection** for each ticker:

```bash
# For each ticker: check cache → fetch if stale → save → compute health
cd $(pwd)/tool_scripts/db_ops && uv run python research_cache_ops.py --is-fresh <ticker> <market> financials
# If stale: fetch + bulk-upsert + compute-health + mark cache (see CLAUDE.md 研究快取 workflow)
# If fresh: read from DB
```

3. **Format comparison table in 繁體中文:**

```
⚖️ 股票比較

| 指標 | <T1> | <T2> | <T3> |
|------|------|------|------|
| 現價 | XX | XX | XX |
| 市值 | XX | XX | XX |
| 本益比 (P/E) | XX | XX | XX |
| 股價淨值比 (P/B) | XX | XX | XX |
| ROE | XX% | XX% | XX% |
| ROA | XX% | XX% | XX% |
| 負債權益比 (D/E) | XX | XX | XX |
| 毛利率 | XX% | XX% | XX% |
| 營業利益率 | XX% | XX% | XX% |
| 淨利率 | XX% | XX% | XX% |
| 自由現金流 | XX | XX | XX |
| 殖利率 | XX% | XX% | XX% |
| Z-Score | XX | XX | XX |
| F-Score | X/9 | X/9 | X/9 |
| 52週高/低 | XX/XX | XX/XX | XX/XX |

🏆 綜合評比：<brief comparison summary>
```

4. Highlight the best value in each row with bold.

5. Report the formatted result to the user.
