# 查詢個股資訊

Look up company profile, sector, and key metrics for a stock.

## Usage

```
/lookup_stock <ticker> [market]
```

## Examples

```
/lookup_stock AAPL
/lookup_stock 2330 TW
/lookup_stock 000858 CN
```

## Instructions

When this command is invoked:

1. **Parse arguments:**
   - `ticker`: Stock ticker symbol (required)
   - `market`: Market code — `US`, `TW`, or `CN` (optional, auto-detect)

2. **Auto-detect market** if not specified (same rules as check_price)

3. **Fetch company info and key metrics** using the market data fetcher:

```bash
cd $(pwd)/tool_scripts/market_data && uv run python fetcher_factory.py info <ticker> --market <market>
cd $(pwd)/tool_scripts/market_data && uv run python fetcher_factory.py metrics <ticker> --market <market>
```

4. **Format output in 繁體中文:**

```
🏢 <公司名稱> (<ticker>)
市場：<market> | 產業：<sector>
市值：<market_cap>
員工數：<employees>

📊 關鍵指標
本益比 (P/E)：<pe_ratio>
股價淨值比 (P/B)：<pb_ratio>
股東權益報酬率 (ROE)：<roe>%
資產報酬率 (ROA)：<roa>%
負債權益比 (D/E)：<de_ratio>
殖利率：<dividend_yield>%
52週高/低：<52w_high> / <52w_low>
```

5. Report the formatted result to the user.
