# 發送報告 HTML

Send a research report in HTML format to Telegram as a document file.

## Usage

```
/send_report_html [report_path]
/send_report_html
```

## Examples

```
/send_report_html output/20260228_2323_cn_3yr_low_research.html
/send_report_html   (sends the most recent HTML report)
```

## Instructions

When this command is invoked:

1. **Determine the HTML report file:**
   - If user provided a path argument, use that file
   - If NO argument is provided, find the most recent `.html` file in `output/` directory:

   ```bash
   LATEST_HTML=$(ls -t output/*.html 2>/dev/null | head -1)
   ```

2. **Verify the HTML file exists:**
   - If file doesn't exist, check if there's a corresponding `.md` file
   - If `.md` exists but `.html` doesn't, offer to convert it first

3. **Send the HTML file as a document to Telegram:**

   ```bash
   cd tool_scripts/send_telegram && uv run python send_message.py \
     --send-file ../../<HTML_FILE_PATH> \
     --caption "📊 CCStockWorkEnv 研究報告 (HTML)"
   ```

4. **Report result to user in 繁體中文:**

   Success:
   ```
   ✅ HTML 報告已發送至 Telegram

   檔案：<filename>.html
   大小：<file_size> KB
   格式：HTML (可在瀏覽器開啟)

   📱 請至 Telegram 查收文件
   ```

   If file not found:
   ```
   ❌ 找不到 HTML 報告檔案

   最近的報告：
   - output/<latest>.md

   💡 建議：
   1. 先使用 /gen_report 產生報告 (會自動產生 HTML 版本)
   2. 或指定完整路徑：/send_report_html output/<filename>.html
   ```

## Notes

- **Markdown to HTML conversion**: The `/gen_report` command with `--format both` generates both `.md` and `.html` versions
- **File format**: Sends the actual HTML file as a Telegram document (not as text message)
- **Supported reports**: single, comparison, screening, sector, cn_3yr_low
- **HTML advantages**: Preserves tables, formatting, and can include embedded charts as base64 images
- **File size limit**: Telegram document limit is 50 MB (reports are typically < 1 MB)

## Related Commands

- `/gen_report` — Generate a new report (creates both .md and .html)
- `/send_telegram` — Send text messages to Telegram
- `/send_email` — Email the HTML report with chart attachments
