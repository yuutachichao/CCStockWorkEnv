# 更新資料庫

Incremental update of daily stock prices in the database.

## Usage

```
/update_db [market] [--full]
```

## Examples

```
/update_db
/update_db US
/update_db TW --full
```

## Arguments

- `market`: `US`, `TW`, or `CN` (optional, default: all markets)
- `--full`: Force full re-download instead of incremental update

## Instructions

When this command is invoked:

1. **Check existing data** to determine what needs updating:

```bash
cd $(pwd)/tool_scripts/db_ops && uv run python price_ops.py --last-date --market <market>
```

2. **Run incremental update** (only fetch data since last update):

```bash
cd $(pwd)/tool_scripts/db_ops && uv run python price_ops.py --update --market <market>
```

Or with `--full` flag:
```bash
cd $(pwd)/tool_scripts/db_ops && uv run python price_ops.py --bulk-download --market <market> --days 365
```

3. **Report progress in 繁體中文:**

```
🔄 更新資料庫
市場：<market or 全部>
模式：<增量更新 / 完整重新下載>

⏳ 更新進度...
✅ 完成
- 更新筆數：<count>
- 最新日期：<date>
- 資料庫大小：<size>
```

4. Report the formatted result to the user.
