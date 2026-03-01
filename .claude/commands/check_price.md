# 查詢即時股價

Fetch current stock price, change, and volume for a given ticker.

## Usage

```
/check_price <ticker> [market]
```

## Examples

```
/check_price TSLA
/check_price 2330 TW
/check_price 600519 CN
```

## Instructions

When this command is invoked:

1. **Parse arguments:**
   - `ticker`: Stock ticker symbol (required)
   - `market`: Market code — `US`, `TW`, or `CN` (optional, auto-detect if not provided)

2. **Auto-detect market** if not specified:
   - Pure digits → `TW` (Taiwan stocks use numeric tickers)
   - 6 digits starting with 6/0/3 → `CN` (Shanghai/Shenzhen)
   - Otherwise → `US`

3. **Fetch price data** using the market data fetcher:

```bash
cd $(pwd)/tool_scripts/market_data && uv run python fetcher_factory.py quote <ticker> --market <market>
```

4. **Format output in 繁體中文:**

```
📊 <公司名稱> (<ticker>)
市場：<market>
現價：<price> <currency>
漲跌：<change> (<change_pct>%)
成交量：<volume>
更新時間：<timestamp>
```

5. Report the formatted result to the user.
