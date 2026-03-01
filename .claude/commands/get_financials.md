# 查看財報

Fetch and display financial statements (income, balance sheet, cash flow) for a stock.

## Usage

```
/get_financials <ticker> [market] [period]
```

## Examples

```
/get_financials TSLA
/get_financials 2330 TW quarterly
/get_financials AAPL US annual
```

## Arguments

- `ticker`: Stock ticker symbol (required)
- `market`: `US`, `TW`, or `CN` (optional, auto-detect)
- `period`: `annual` or `quarterly` (default: `annual`)

## Instructions

When this command is invoked:

1. **Parse arguments** and auto-detect market if needed

2. **Cache-first data collection:**

```bash
# Check if financials cache is fresh
cd $(pwd)/tool_scripts/db_ops && uv run python research_cache_ops.py --is-fresh <ticker> <market> financials

# If stale or missing: fetch + save
cd $(pwd)/tool_scripts/market_data && uv run python fetcher_factory.py financials <ticker> --market <market> --period <period>
cd $(pwd)/tool_scripts/db_ops && uv run python financial_ops.py --bulk-upsert --json '<json>'
cd $(pwd)/tool_scripts/db_ops && uv run python research_cache_ops.py --mark <ticker> <market> financials

# Read from DB
cd $(pwd)/tool_scripts/db_ops && uv run python financial_ops.py --get <ticker> --market <market> --period <period>
```

3. **Format output in 繁體中文** with key financial data:

```
📋 <公司名稱> (<ticker>) 財務報表 — <period>

💰 損益表
營收：<revenue>
毛利：<gross_profit> (毛利率 <margin>%)
營業利益：<operating_income> (營業利益率 <margin>%)
淨利：<net_income> (淨利率 <margin>%)
每股盈餘 (EPS)：<eps>

📊 資產負債表
總資產：<total_assets>
總負債：<total_liabilities>
股東權益：<total_equity>
流動資產：<current_assets>
流動負債：<current_liabilities>

💵 現金流量表
營業現金流：<operating_cf>
資本支出：<capex>
自由現金流：<fcf>

📈 年增率 (YoY)
營收成長：<revenue_growth>%
淨利成長：<net_income_growth>%
EPS 成長：<eps_growth>%
```

4. If data spans multiple periods, show the most recent 4 periods in a table format.

5. Report the formatted result to the user.
