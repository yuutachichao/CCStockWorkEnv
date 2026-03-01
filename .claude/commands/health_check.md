# 健康檢查

Run a comprehensive financial health check on a stock: Z-Score, F-Score, key ratios, and overall verdict.

## Usage

```
/health_check <ticker> [market]
```

## Examples

```
/health_check AAPL
/health_check 2330 TW
/health_check 600519 CN
```

## Instructions

When this command is invoked:

1. **Parse arguments** and auto-detect market if needed

2. **Cache-first data collection:**

```bash
# Check if financials cache is fresh
cd $(pwd)/tool_scripts/db_ops && uv run python research_cache_ops.py --is-fresh <ticker> <market> financials

# If stale or missing:
cd $(pwd)/tool_scripts/market_data && uv run python fetcher_factory.py financials <ticker> --market <market> --period annual
cd $(pwd)/tool_scripts/db_ops && uv run python financial_ops.py --bulk-upsert --json '<json>'
cd $(pwd)/tool_scripts/db_ops && uv run python financial_ops.py --compute-health <ticker> --market <market>
cd $(pwd)/tool_scripts/db_ops && uv run python research_cache_ops.py --mark <ticker> <market> financials

# If fresh: read from DB
cd $(pwd)/tool_scripts/db_ops && uv run python financial_ops.py --get-health <ticker> --market <market>
```

Also fetch key ratios for current valuation:
```bash
cd $(pwd)/tool_scripts/financial_calc && uv run python ratios.py <ticker> --market <market>
```

3. **Format output in 繁體中文:**

```
🏥 健康檢查：<公司名稱> (<ticker>)

📊 Altman Z-Score：<score>
判定：<安全區 / 灰色地帶 / 危險區>
- 安全區 (>2.99)：破產風險極低
- 灰色地帶 (1.81-2.99)：需留意
- 危險區 (<1.81)：破產風險高

📈 Piotroski F-Score：<score>/9
判定：<財務強健 / 普通 / 疲弱>
✅ 獲利能力：<X>/4
  - ROA 正值：<✅/❌>
  - 營業現金流正值：<✅/❌>
  - ROA 年增：<✅/❌>
  - 現金流 > 淨利：<✅/❌>
✅ 槓桿/流動性：<X>/3
  - 長期負債下降：<✅/❌>
  - 流動比率上升：<✅/❌>
  - 未發新股：<✅/❌>
✅ 營運效率：<X>/2
  - 毛利率上升：<✅/❌>
  - 資產週轉率上升：<✅/❌>

📋 關鍵比率
| 指標 | 數值 | 評等 |
|------|------|------|
| 本益比 (P/E) | XX | ⭐⭐⭐ |
| 股價淨值比 (P/B) | XX | ⭐⭐⭐ |
| ROE | XX% | ⭐⭐⭐ |
| 負債權益比 (D/E) | XX | ⭐⭐⭐ |
| 流動比率 | XX | ⭐⭐⭐ |
| 自由現金流收益率 | XX% | ⭐⭐⭐ |

🎯 綜合評價：<verdict>
```

4. Report the formatted result to the user.
