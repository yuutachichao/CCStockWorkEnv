# 產生報告

Generate a mobile-friendly research report, verify it in Chrome browser, then send the URL to Telegram.

**IMPORTANT**: Follow `.claude/skills/report_generation_guide.md` for all HTML structure, CSS, and JS requirements.

## Usage

```
/gen_report <type> [tickers] [--email] [--no-telegram]
```

## Types

- `single <ticker>` — Single stock deep dive report
- `screening <criteria>` — Screening results report
- `comparison <t1> <t2> [t3...]` — Stock comparison report
- `sector <market> <sector>` — Sector overview report
- `cn_3yr_low` — China A-share 3-year low research report

## Examples

```
/gen_report single TSLA
/gen_report screening P/E < 15 且 ROE > 15%
/gen_report comparison AAPL MSFT GOOGL
/gen_report cn_3yr_low --email
```

## Instructions

When this command is invoked:

1. **Parse report type and arguments**

2. **Collect data (cache-first)** — for each stock, check cache freshness first:

```bash
# Per ticker: check → fetch if stale → save → compute health → mark
cd $(pwd)/tool_scripts/db_ops && uv run python research_cache_ops.py --is-fresh <ticker> <market> financials
# Follow CLAUDE.md 研究快取 workflow for stale/missing data
```

3. **Generate charts** as needed:

```bash
cd tool_scripts/report_gen && uv run python chart_gen.py --type <chart_type> --ticker <ticker> --market <market> --output ../../data/charts/
```

4. **Generate report HTML file** following the report_generation_guide skill:

- Output to `output/YYYYMMDD_HHMM_<type>.html`
- Must include full CSS from the guide (viewport meta, RWD media queries, table-container, etc.)
- Must include JS from the guide (collapsible sections, scroll hints)
- All tables wrapped in `<div class="table-container">`
- Summary stats use `.stats-grid` layout
- Long sections use `.collapsible` + `.collapsible-content`
- Color coding: `.text-safe` (green), `.text-grey` (orange), `.text-danger` (red)

For cn_3yr_low type, can also use the Python generator:
```bash
cd tool_scripts/report_gen && uv run python cn_3yr_low_report.py --format both
```

For other types:
```bash
cd tool_scripts/report_gen && uv run python markdown_report.py --type <type> --tickers <tickers> --output ../../output/
```

5. **Ensure web server is running:**

```bash
if ! curl -s http://localhost:8800 > /dev/null 2>&1; then
    cd tool_scripts/web_server && bash start_server.sh
    sleep 2
fi
```

6. **Browser verification (MANDATORY):**

This step is required. Do NOT skip it.

```
6.1 Get tab context:
    → mcp__claude-in-chrome__tabs_context_mcp (createIfEmpty: true)

6.2 Create a new tab:
    → mcp__claude-in-chrome__tabs_create_mcp

6.3 Navigate to report:
    → mcp__claude-in-chrome__navigate (url: http://localhost:8800/reports/<slug>/)

6.4 Verify MOBILE view:
    → mcp__claude-in-chrome__resize_window (width: 375, height: 812)
    → mcp__claude-in-chrome__computer (action: screenshot)
    → Check:
       - Title displays correctly, no truncation
       - Tables have scroll hint visible
       - Cards and badges render properly
       - Text is readable (≥ 13px)
       - No horizontal overflow on body
       - Color coding is correct (green/orange/red)

6.5 Verify DESKTOP view:
    → mcp__claude-in-chrome__resize_window (width: 1280, height: 800)
    → mcp__claude-in-chrome__computer (action: screenshot)
    → Check: layout is centered, max-width working, tables readable

6.6 If verification FAILS:
    → Fix the HTML file
    → Re-verify (go back to 6.3)
    → Do NOT proceed to step 7 until verification passes

6.7 If verification PASSES:
    → Proceed to step 7
```

If Chrome browser extension is not connected, fall back to curl-based validation:
```bash
# Check HTML structure
curl -s http://localhost:8800/reports/<slug>/ | grep -c "viewport"        # must be ≥ 1
curl -s http://localhost:8800/reports/<slug>/ | grep -c "table-container"  # must match table count
curl -s http://localhost:8800/reports/<slug>/ | grep -c "scroll-hint"      # should exist
curl -s http://localhost:8800/reports/<slug>/ | grep -c "stats-grid"       # should exist for summaries
```

7. **Send report link to Telegram (unless `--no-telegram`):**

```python
import json
with open('config.json') as f:
    config = json.load(f)

ws = config.get('web_server', {})
fixed_ip = ws.get('fixed_ip') or None
port = ws.get('external_port', ws.get('port', 8800))
slug = "<timestamp>_<type>"

if fixed_ip:
    url = f"http://{fixed_ip}:{port}/reports/{slug}/"
else:
    # No public IP configured — use localhost and warn user
    internal_port = ws.get('internal_port', ws.get('port', 8800))
    url = f"http://localhost:{internal_port}/reports/{slug}/"
    # WARN: this URL is only accessible on the local network
```

```bash
cd tool_scripts/send_telegram && uv run python send_message.py \
  --message "📊 <b>報告已產生</b>

類型: {report_display_name}
時間: {formatted_time}

🔗 <a href='{url}'>點此瀏覽報告</a>

💡 點擊連結在手機瀏覽器開啟"
```

8. **If `--email` is specified**, send HTML report with chart attachments:

```bash
cd tool_scripts/send_mail && uv run python send_mail.py \
  --subject "CCStockWorkEnv 報告：<report_type>" \
  --html-file ../../output/<timestamp>_<report>.html \
  --body "CCStockWorkEnv 研究報告（見 HTML 版本）" \
  --attachment ../../data/charts/<chart1>.png \
  --attachment ../../data/charts/<chart2>.png
```

9. **Reply to the user in 繁體中文:**

```
📝 報告已產生 ✅ 驗證通過

類型：<report_type>
時間：<formatted_time>

🔗 網址：http://<fixed_ip>:<port>/reports/<slug>/

報告摘要：
<brief summary of key findings>

✅ 已透過手機模式驗證
✅ Telegram 已收到連結
💡 點擊連結在手機瀏覽器開啟效果最佳
```
