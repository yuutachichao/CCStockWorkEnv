# 篩選股票

Screen stocks based on financial criteria using natural language.

## Usage

```
/screen_stocks <criteria>
```

## Examples

```
/screen_stocks P/E < 15 且 ROE > 15%
/screen_stocks 台股中殖利率 > 5% 的股票
/screen_stocks US stocks with market cap > 10B and D/E < 0.5
/screen_stocks 找出三年低點的中國A股
```

## Instructions

When this command is invoked:

1. **Parse natural language criteria** into structured filters:
   - Identify market (US/TW/CN) from context
   - Extract metric conditions (P/E, ROE, P/B, D/E, yield, market cap, etc.)
   - Detect special patterns like "N-year low", "near 52-week low", etc.

2. **Run the screener:**

```bash
cd $(pwd)/tool_scripts/financial_calc && uv run python screener.py --market <market> --criteria '<json_criteria>'
```

Where `json_criteria` is a JSON string like:
```json
{"market": "US", "filters": [{"metric": "pe_ratio", "op": "<", "value": 15}, {"metric": "roe", "op": ">", "value": 0.15}]}
```

3. **Format output in 繁體中文:**

```
🔍 篩選條件：<human-readable criteria>
市場：<market>
符合條件：<count> 檔

| # | 代號 | 名稱 | 現價 | P/E | ROE | <relevant metrics> |
|---|------|------|------|-----|-----|--------------------|
| 1 | XXXX | XXX  | XX.X | XX  | XX% | ...                |
...

📊 共 <count> 檔符合條件
```

4. If results exceed 20, show top 20 and mention total count.

5. Report the formatted result to the user.
