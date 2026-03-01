# Scheduler SOP — 排程任務設定標準流程

當使用者要求「每天執行某任務」、「定時跑報告」等排程需求時,遵循本 SOP 設定。

macOS 使用 `launchd` 管理排程 (不使用 cron)。

---

## 排程架構

```
launchd plist (定時觸發)
    → shell script (任務包裝)
        → uv run python <tool_script> (實際執行)
        → send_telegram (結果通知)
    → log files (執行紀錄)
```

所有排程相關檔案統一放在:

```
CCStockWorkEnv/
├── schedules/                          # 排程腳本目錄
│   ├── daily_cn_3yr_low.sh             # 範例: 每日中國A股低點掃描
│   ├── daily_price_update.sh           # 範例: 每日價格更新
│   └── ...
├── data/
│   └── logs/                           # 排程日誌目錄
│       ├── daily_cn_3yr_low.log
│       ├── daily_cn_3yr_low.error.log
│       └── ...
```

plist 放在:
```
~/Library/LaunchAgents/com.ccstockworkenv.<task_name>.plist
```

---

## Step 1: 建立排程腳本

在 `schedules/` 目錄建立 shell script。

### 腳本模板

```bash
#!/bin/bash
# ==================================================
# CCStockWorkEnv Scheduled Task: <任務描述>
# Schedule: <執行頻率,例如 每日 09:00>
# ==================================================

set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
PROJECT_ROOT="$(cd "$SCRIPT_DIR/.." && pwd)"
TASK_NAME="<task_name>"
LOG_FILE="$PROJECT_ROOT/data/logs/${TASK_NAME}.log"

# Ensure log directory exists
mkdir -p "$PROJECT_ROOT/data/logs"

# Timestamp
echo "========================================" >> "$LOG_FILE"
echo "[$(date '+%Y-%m-%d %H:%M:%S')] Starting $TASK_NAME" >> "$LOG_FILE"

# ---------- Task execution ----------

cd "$PROJECT_ROOT/tool_scripts/<subfolder>"
RESULT=$(uv run python <script>.py <args> 2>&1) || {
    # On failure: notify via Telegram
    cd "$PROJECT_ROOT/tool_scripts/send_telegram"
    uv run python send_message.py --message "❌ <b>排程任務失敗</b>

任務: $TASK_NAME
時間: $(date '+%Y-%m-%d %H:%M')
錯誤: 請查看日誌"

    echo "[$(date '+%Y-%m-%d %H:%M:%S')] FAILED" >> "$LOG_FILE"
    echo "$RESULT" >> "$LOG_FILE"
    exit 1
}

echo "$RESULT" >> "$LOG_FILE"

# ---------- Send result to Telegram ----------

cd "$PROJECT_ROOT/tool_scripts/send_telegram"
uv run python send_message.py --message "✅ <b>排程任務完成</b>

任務: $TASK_NAME
時間: $(date '+%Y-%m-%d %H:%M')

<結果摘要>"

echo "[$(date '+%Y-%m-%d %H:%M:%S')] Completed successfully" >> "$LOG_FILE"
```

### 重要規則

1. 使用 `set -euo pipefail` — 任何錯誤立即終止
2. 所有路徑使用絕對路徑
3. 成功和失敗都要發 Telegram 通知
4. 所有輸出寫入 log file
5. 腳本必須設為可執行: `chmod +x schedules/<script>.sh`

---

## Step 2: 建立 launchd plist

在 `~/Library/LaunchAgents/` 建立 plist 檔案。

### plist 模板

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.ccstockworkenv.<task_name></string>

    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>/Users/wanghsuanchung/Projects/CCStockWorkEnv/schedules/<task_name>.sh</string>
    </array>

    <key>WorkingDirectory</key>
    <string>/Users/wanghsuanchung/Projects/CCStockWorkEnv</string>

    <!-- Option A: Run at specific time every day -->
    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>9</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>

    <!-- Option B: Run every N seconds (use EITHER A or B, not both) -->
    <!-- <key>StartInterval</key> -->
    <!-- <integer>3600</integer> -->

    <key>StandardOutPath</key>
    <string>/Users/wanghsuanchung/Projects/CCStockWorkEnv/data/logs/<task_name>.log</string>

    <key>StandardErrorPath</key>
    <string>/Users/wanghsuanchung/Projects/CCStockWorkEnv/data/logs/<task_name>.error.log</string>

    <key>EnvironmentVariables</key>
    <dict>
        <key>PATH</key>
        <string>/Users/wanghsuanchung/.local/bin:/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin</string>
    </dict>
</dict>
</plist>
```

### StartCalendarInterval 常用設定

```xml
<!-- 每天 09:00 -->
<key>StartCalendarInterval</key>
<dict>
    <key>Hour</key>
    <integer>9</integer>
    <key>Minute</key>
    <integer>0</integer>
</dict>

<!-- 每週一 09:00 -->
<key>StartCalendarInterval</key>
<dict>
    <key>Weekday</key>
    <integer>1</integer>
    <key>Hour</key>
    <integer>9</integer>
    <key>Minute</key>
    <integer>0</integer>
</dict>

<!-- 每週一到五 09:00 (多個時間點) -->
<key>StartCalendarInterval</key>
<array>
    <dict><key>Weekday</key><integer>1</integer><key>Hour</key><integer>9</integer><key>Minute</key><integer>0</integer></dict>
    <dict><key>Weekday</key><integer>2</integer><key>Hour</key><integer>9</integer><key>Minute</key><integer>0</integer></dict>
    <dict><key>Weekday</key><integer>3</integer><key>Hour</key><integer>9</integer><key>Minute</key><integer>0</integer></dict>
    <dict><key>Weekday</key><integer>4</integer><key>Hour</key><integer>9</integer><key>Minute</key><integer>0</integer></dict>
    <dict><key>Weekday</key><integer>5</integer><key>Hour</key><integer>9</integer><key>Minute</key><integer>0</integer></dict>
</array>

<!-- 每小時 -->
<key>StartInterval</key>
<integer>3600</integer>

<!-- 每30分鐘 -->
<key>StartInterval</key>
<integer>1800</integer>
```

### Weekday 值

| 值 | 星期 |
|----|------|
| 0 | 週日 |
| 1 | 週一 |
| 2 | 週二 |
| 3 | 週三 |
| 4 | 週四 |
| 5 | 週五 |
| 6 | 週六 |

---

## Step 3: 啟用排程

**注意**: `launchctl load` 必須在 Terminal.app 中執行,無法從 Claude Code 環境執行 (macOS 安全限制)。

建立好 plist 和腳本後,告知使用者執行:

```bash
launchctl load ~/Library/LaunchAgents/com.ccstockworkenv.<task_name>.plist
```

### 驗證排程已載入

```bash
launchctl list | grep ccstockworkenv
```

### 手動觸發測試

```bash
# 直接執行腳本測試
bash schedules/<task_name>.sh

# 或透過 launchctl 觸發
launchctl start com.ccstockworkenv.<task_name>
```

---

## Step 4: 管理排程

### 查看所有 CCStockWorkEnv 排程

```bash
launchctl list | grep ccstockworkenv
```

### 停止排程

```bash
launchctl unload ~/Library/LaunchAgents/com.ccstockworkenv.<task_name>.plist
```

### 重新啟動排程

```bash
launchctl unload ~/Library/LaunchAgents/com.ccstockworkenv.<task_name>.plist
launchctl load ~/Library/LaunchAgents/com.ccstockworkenv.<task_name>.plist
```

### 刪除排程

```bash
launchctl unload ~/Library/LaunchAgents/com.ccstockworkenv.<task_name>.plist
rm ~/Library/LaunchAgents/com.ccstockworkenv.<task_name>.plist
rm schedules/<task_name>.sh
```

### 查看日誌

```bash
# 最近的執行紀錄
tail -50 data/logs/<task_name>.log

# 錯誤日誌
cat data/logs/<task_name>.error.log

# 即時追蹤
tail -f data/logs/<task_name>.log
```

---

## 完整範例: 每日中國A股三年低點掃描

### 使用者需求

> 「每天早上九點自動跑一次中國A股三年低點掃描,結果發到 Telegram」

### 1. 建立腳本

**檔案**: `schedules/daily_cn_3yr_low.sh`

```bash
#!/bin/bash
set -euo pipefail

PROJECT_ROOT="/Users/wanghsuanchung/Projects/CCStockWorkEnv"
TASK_NAME="daily_cn_3yr_low"
LOG_FILE="$PROJECT_ROOT/data/logs/${TASK_NAME}.log"
TIMESTAMP=$(date '+%Y%m%d_%H%M')

mkdir -p "$PROJECT_ROOT/data/logs"

echo "========================================" >> "$LOG_FILE"
echo "[$(date '+%Y-%m-%d %H:%M:%S')] Starting $TASK_NAME" >> "$LOG_FILE"

# Generate report
cd "$PROJECT_ROOT/tool_scripts/report_gen"
uv run python cn_3yr_low_report.py --format html >> "$LOG_FILE" 2>&1 || {
    cd "$PROJECT_ROOT/tool_scripts/send_telegram"
    uv run python send_message.py --message "❌ <b>每日掃描失敗</b>

任務: 中國A股三年低點掃描
時間: $(date '+%Y-%m-%d %H:%M')
錯誤: 報告產生失敗,請查看日誌"
    exit 1
}

# Find the generated report
REPORT_FILE=$(ls -t "$PROJECT_ROOT/output/"*cn_3yr_low*.html 2>/dev/null | head -1)

if [ -z "$REPORT_FILE" ]; then
    cd "$PROJECT_ROOT/tool_scripts/send_telegram"
    uv run python send_message.py --message "❌ <b>每日掃描失敗</b>

任務: 中國A股三年低點掃描
時間: $(date '+%Y-%m-%d %H:%M')
錯誤: 找不到產生的報告檔案"
    exit 1
fi

# Extract slug from filename
SLUG=$(basename "$REPORT_FILE" .html)

# Read config for web server URL
FIXED_IP=$(python3 -c "
import json
with open('$PROJECT_ROOT/config.json') as f:
    ws = json.load(f).get('web_server', {})
ip = ws.get('fixed_ip') or 'localhost'
print(ip)
")
PORT=$(python3 -c "
import json
with open('$PROJECT_ROOT/config.json') as f:
    ws = json.load(f).get('web_server', {})
print(ws.get('external_port', ws.get('port', 8800)))
")

URL="http://${FIXED_IP}:${PORT}/reports/${SLUG}/"

# Send to Telegram
cd "$PROJECT_ROOT/tool_scripts/send_telegram"
uv run python send_message.py --message "📊 <b>每日掃描完成</b>

任務: 中國A股三年低點掃描
時間: $(date '+%Y-%m-%d %H:%M')

🔗 <a href='${URL}'>點此瀏覽報告</a>

💡 點擊連結在手機瀏覽器開啟"

echo "[$(date '+%Y-%m-%d %H:%M:%S')] Completed. Report: $SLUG" >> "$LOG_FILE"
```

### 2. 建立 plist

**檔案**: `~/Library/LaunchAgents/com.ccstockworkenv.daily_cn_3yr_low.plist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.ccstockworkenv.daily_cn_3yr_low</string>

    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>/Users/wanghsuanchung/Projects/CCStockWorkEnv/schedules/daily_cn_3yr_low.sh</string>
    </array>

    <key>WorkingDirectory</key>
    <string>/Users/wanghsuanchung/Projects/CCStockWorkEnv</string>

    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>9</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>

    <key>StandardOutPath</key>
    <string>/Users/wanghsuanchung/Projects/CCStockWorkEnv/data/logs/daily_cn_3yr_low.log</string>

    <key>StandardErrorPath</key>
    <string>/Users/wanghsuanchung/Projects/CCStockWorkEnv/data/logs/daily_cn_3yr_low.error.log</string>

    <key>EnvironmentVariables</key>
    <dict>
        <key>PATH</key>
        <string>/Users/wanghsuanchung/.local/bin:/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin</string>
    </dict>
</dict>
</plist>
```

### 3. 啟用

```bash
chmod +x schedules/daily_cn_3yr_low.sh
launchctl load ~/Library/LaunchAgents/com.ccstockworkenv.daily_cn_3yr_low.plist
```

### 4. 測試

```bash
# 手動執行一次
bash schedules/daily_cn_3yr_low.sh

# 查看日誌
tail -20 data/logs/daily_cn_3yr_low.log
```

---

## Telegram 通知格式

### 成功通知

```
✅ <b>排程任務完成</b>

任務: <任務名稱>
時間: YYYY-MM-DD HH:MM

<結果摘要或報告連結>
```

### 失敗通知

```
❌ <b>排程任務失敗</b>

任務: <任務名稱>
時間: YYYY-MM-DD HH:MM
錯誤: <簡要錯誤描述>
```

---

## 注意事項

1. **時區**: launchd 使用系統時區 (Asia/Taipei, UTC+8)
2. **登入狀態**: LaunchAgents 只在使用者登入時執行。如果電腦休眠或登出,排程會在下次登入時補執行
3. **PATH**: plist 的 EnvironmentVariables 必須包含 `uv` 和 `python` 的路徑
4. **launchctl 限制**: `launchctl load/unload` 必須在 Terminal.app 執行,Claude Code 環境無法執行。建立好檔案後,提示使用者在 Terminal.app 中執行載入指令
5. **日誌清理**: 日誌檔案會持續增長,建議定期清理或設定 logrotate
