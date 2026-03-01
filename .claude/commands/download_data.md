# 下載數據

Batch download stock data from a market and store in SQLite database.

## Usage

```
/download_data <market> [sector] [--days N]
```

## Examples

```
/download_data US
/download_data TW --days 365
/download_data CN technology
```

## Arguments

- `market`: `US`, `TW`, or `CN` (required)
- `sector`: Filter by sector (optional)
- `--days N`: Number of days of historical data (default: 365)

## Instructions

When this command is invoked:

1. **Initialize database** if not exists:

```bash
cd $(pwd)/tool_scripts/db_ops && uv run python db_manager.py --init
```

2. **Get stock list** for the market:

```bash
cd $(pwd)/tool_scripts/market_data && uv run python fetcher_factory.py list-tickers --market <market> [--sector <sector>]
```

3. **Download price data** for each stock and store in DB:

```bash
cd $(pwd)/tool_scripts/db_ops && uv run python price_ops.py --bulk-download --market <market> --days <days>
```

4. **Report progress in 繁體中文:**

```
📥 下載數據：<market> 市場
產業篩選：<sector or 全部>
期間：最近 <days> 天

⏳ 下載進度...
✅ 完成：<count> 檔股票已下載
❌ 失敗：<error_count> 檔
💾 資料庫：data/ccstockworkenv.db
```

5. Report the formatted result to the user.
