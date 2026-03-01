# Chart Generation — 圖表產生指南

## Available Chart Types

### 1. Price History Chart
Line chart with volume bars showing price movement over time.

```bash
cd tool_scripts/report_gen && uv run python chart_gen.py \
    --type price --ticker AAPL --market US --days 365 --output ../../data/charts/
```

Output: `AAPL_US_price_20260228.png`

### 2. Comparison Chart
Normalized line chart comparing multiple stocks (base=100).

```bash
cd tool_scripts/report_gen && uv run python chart_gen.py \
    --type comparison --tickers AAPL,MSFT,GOOGL --market US --output ../../data/charts/
```

Output: `comparison_AAPL_MSFT_GOOGL_20260228.png`

### 3. Financials Chart
Grouped bar chart showing Revenue, Net Income, and FCF trends.

```bash
cd tool_scripts/report_gen && uv run python chart_gen.py \
    --type financials --ticker AAPL --market US --output ../../data/charts/
```

Output: `AAPL_US_financials_20260228.png`

### 4. Radar Chart
Spider/radar chart showing Opportunity Score breakdown (Value, Quality, Safety, Momentum, Income).

Generated programmatically via `chart_gen.generate_radar_chart()`.

Output: `AAPL_radar_20260228.png`

## Output Paths

- **Charts**: `data/charts/` (gitignored)
- **Reports**: `output/` (gitignored)
- **Format**: PNG, 150 DPI

## Chart Style

- Non-interactive backend (`matplotlib.use("Agg")`)
- Color palette: Material Design colors
  - Primary: `#2196F3` (Blue)
  - Secondary: `#F44336` (Red)
  - Success: `#4CAF50` (Green)
  - Warning: `#FF9800` (Orange)
  - Accent: `#9C27B0` (Purple)
- Figure size: 12x6 (landscape) or 12x8 (with volume)
- Grid: alpha=0.3
- Font: System default (supports CJK characters)

## Sending Charts via Telegram

After generating a chart:
```bash
cd tool_scripts/send_telegram && uv run python send_message.py \
    --send-file ../../data/charts/AAPL_US_price_20260228.png \
    --caption "AAPL 價格走勢圖"
```

## Dependencies

- `matplotlib>=3.8.0` (in report_gen/pyproject.toml)
- `numpy` (for radar chart)
- CJK font support for Chinese labels (matplotlib auto-detects system fonts)
