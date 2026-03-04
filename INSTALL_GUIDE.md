# CCStockWorkEnv — 全自動化安裝與移植技術文件

> **文件目的**：讓你在全新 Windows 環境（重灌或換電腦）時，以最快速度完成完整移植。
> **適用平台**：Windows 11 + Claude Desktop
> **最後更新**：2026-03-03

---

## 目錄

1. [環境變數鎖定](#1-環境變數鎖定)
2. [標準化安裝流程](#2-標準化安裝流程)
3. [環境移植關鍵](#3-環境移植關鍵)
4. [坑點防禦](#4-坑點防禦)
5. [因果驗證清單](#5-因果驗證清單)
6. [依賴清單](#6-依賴清單)

---

## 1. 環境變數鎖定

### 硬體需求

| 項目 | 最低需求 | 備註 |
|------|----------|------|
| RAM | 8 GB | 16 GB 建議（yfinance 批次抓取時） |
| 儲存空間 | 5 GB 可用 | `.venv` × 7 個子模組，每個約 200~500 MB |
| 網路 | 穩定網際網路 | 抓取市場數據必須 |
| GPU | 不需要 | 純 CPU Python 環境 |

### 作業系統

| 項目 | 版本 |
|------|------|
| Windows | 11（10 應可用，未完整測試） |
| 架構 | x86-64（ARM 未測試） |

### 必備基礎軟體

| 軟體 | 版本 | 用途 | 安裝來源 |
|------|------|------|----------|
| **Git for Windows** | 2.x+ | 版本控制 + 提供 Git Bash（`bash.exe`） | https://git-scm.com/download/win |
| **Python** | 3.10+ | 執行所有工具腳本 | https://www.python.org/downloads/ |
| **uv** | 0.10+ | Python 套件與虛擬環境管理 | 見下方 |
| **Claude Desktop** | 最新版 | Claude Code 宿主環境 | https://claude.ai/download |

> ⚠️ **安裝 Python 時**：勾選 `Add Python to PATH`，否則 uv 找不到 Python。

---

## 2. 標準化安裝流程

### Step 0 — 安裝基礎軟體

```powershell
# 0-1. 安裝 Git for Windows（選擇「Use Git Bash as default shell」）
# 手動下載：https://git-scm.com/download/win

# 0-2. 安裝 Python 3.12（勾選 Add to PATH）
# 手動下載：https://www.python.org/downloads/

# 0-3. 安裝 uv（在 PowerShell 執行）
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"

# 0-4. 確認 uv 安裝位置（記下這個路徑，後續設定要用）
where uv
# 預期輸出類似：C:\Users\<你的帳號>\AppData\Local\uv\bin\uv.exe
# 或：C:\Users\<你的帳號>\AppData\Local\Python\pythoncore-3.14-64\Scripts\uv.exe
```

**驗證 Step 0**：
```powershell
git --version    # 預期：git version 2.x.x
python --version # 預期：Python 3.1x.x
uv --version     # 預期：uv 0.x.x
```

---

### Step 1 — Clone 專案

```bash
# 在 Git Bash 執行（不要用 PowerShell）
git clone https://github.com/yuutachichao/CCStockWorkEnv.git
cd CCStockWorkEnv
```

**驗證 Step 1**：
```bash
ls CLAUDE.md tool_scripts/ .claude/
# 預期看到三個項目都存在
```

---

### Step 2 — 還原機密設定檔

```bash
# 從備份複製 config.json（含 Telegram Token）
cp /path/to/backup/config.json config.json

# 驗證格式正確（應輸出 bot_token 欄位）
python -c "import json; c=json.load(open('config.json')); print('Token OK:', bool(c['telegram']['bot_token']))"
```

若無備份，從範本重新建立：
```bash
cp config.json.template config.json
# 用文字編輯器填入：
# - telegram.bot_token
# - telegram.chat_ids
# - web_server.fixed_ip（有公網 IP 才填）
```

**驗證 Step 2**：
```bash
python -c "import json; json.load(open('config.json')); print('config.json OK')"
```

---

### Step 3 — 修正 Claude Code Shell 設定（Windows 必做）

這是 Windows 環境最關鍵的一步。Claude Code 的 Bash 工具需要 POSIX shell，但 Windows 預設的 SHELL 環境變數可能指向 `git.exe`（非 shell），導致所有 Bash 指令失敗。

```powershell
# 在 PowerShell 建立（或覆寫）Claude 全域設定
$settingsDir = "$env:USERPROFILE\.claude"
New-Item -ItemType Directory -Force -Path $settingsDir | Out-Null

$uvPath = (Get-Command uv -ErrorAction SilentlyContinue).Path
if (-not $uvPath) {
    # uv 不在 PATH，手動找
    $uvPath = Get-ChildItem "$env:USERPROFILE\AppData\Local" -Recurse -Name "uv.exe" -ErrorAction SilentlyContinue | Select-Object -First 1
    $uvPath = "$env:USERPROFILE\AppData\Local\$uvPath"
}
# 轉換為 Git Bash 路徑格式（C:\Users\... → /c/Users/...）
$uvGitPath = $uvPath -replace "\\", "/" -replace "^([A-Z]):", { "/$(($_.Groups[1].Value).ToLower())" }

$settings = @{
    env = @{
        SHELL = "C:/Program Files/Git/bin/bash.exe"
        PATH  = "$uvGitPath" -replace "/uv.exe", "" | ForEach-Object { "$_`:/usr/local/bin:/usr/bin:/bin:/mingw64/bin" }
    }
} | ConvertTo-Json -Depth 3

$settings | Set-Content "$settingsDir\settings.json" -Encoding UTF8
Write-Host "Claude settings written to: $settingsDir\settings.json"
```

或手動建立 `C:\Users\<你的帳號>\.claude\settings.json`，內容如下：

```json
{
  "env": {
    "SHELL": "C:/Program Files/Git/bin/bash.exe",
    "PATH": "/c/Users/<你的帳號>/AppData/Local/uv/bin:/usr/local/bin:/usr/bin:/bin:/mingw64/bin"
  }
}
```

> ⚠️ **修改 settings.json 後必須完全重啟 Claude Desktop 才能生效！**
> 重啟方式：系統匣 → 右鍵 Claude 圖示 → Quit → 重新開啟

**驗證 Step 3**（重啟 Claude Desktop 後，在 Claude Code 中執行）：
```bash
echo "shell ok" && which uv
# 預期輸出：shell ok
#           /c/Users/.../uv.exe（或顯示 uv 路徑）
```

---

### Step 4 — 初始化資料庫

```bash
UV="$(which uv)"   # 若 uv 不在 PATH，改用完整路徑

# 初始化 SQLite DB（建立所有資料表）
cd tool_scripts/db_ops && "$UV" run python db_manager.py --init

# 手動補上 v2 欄位（修正已知的 schema 初始化 bug）
"$UV" run python -c "
import sqlite3, os
db = os.path.join('..', '..', 'data', 'ccstockworkenv.db')
conn = sqlite3.connect(db)
existing = {row[1] for row in conn.execute('PRAGMA table_info(financials)').fetchall()}
new_cols = [
    ('quick_ratio','REAL'), ('interest_coverage','REAL'),
    ('ps_ratio','REAL'), ('ev_ebitda','REAL'), ('payout_ratio','REAL'),
    ('cash_and_equivalents','REAL'), ('total_debt','REAL'), ('ebitda','REAL'),
    ('inventory','REAL'), ('receivables','REAL'),
    ('inventory_turnover_days','REAL'), ('receivable_turnover_days','REAL'),
]
added = []
for col, typ in new_cols:
    if col not in existing:
        conn.execute(f'ALTER TABLE financials ADD COLUMN {col} {typ}')
        added.append(col)
conn.commit()
conn.close()
print(f'Added {len(added)} columns: {added}' if added else 'All columns already exist.')
"
```

**驗證 Step 4**：
```bash
"$UV" run python db_manager.py --info
# 預期輸出：Schema version: 2
#           Tables: daily_prices: 0 rows, financials: 0 rows, ...
```

---

### Step 5 — 啟動網頁伺服器

> `start_server.sh` 在 Windows 上不可用（使用 `nohup`、`ps` 等 Unix 指令）。請改用以下方式。

```bash
UV="/c/Users/<你的帳號>/AppData/Local/uv/bin/uv.exe"

# 執行 Django migrations
cd tool_scripts/web_server && "$UV" run python manage.py migrate --run-syncdb

# 在背景啟動伺服器（Git Bash）
nohup "$UV" run python manage.py runserver 0.0.0.0:8800 \
  > ../../data/webserver.log 2>&1 &
echo "Server PID: $!"
```

**驗證 Step 5**：
```bash
sleep 3 && curl -s -o /dev/null -w "%{http_code}" http://localhost:8800/
# 預期輸出：200
```

---

### Step 6 — 設定 Git 身份並推送

```bash
git config --global user.name "yuutachichao"
git config --global user.email "yuutachichao@users.noreply.github.com"

# 確認 remote 指向你的 fork
git remote -v
# 預期：origin  https://github.com/yuutachichao/CCStockWorkEnv.git
```

---

### Step 7 — 功能測試

在 Claude Desktop 的 Claude Code 工作區中，測試以下指令：

```
/check_price AAPL
/check_price 2330 TW
/health_check AAPL
```

---

## 3. 環境移植關鍵

### 必須手動備份的檔案（gitignore 排除）

| 檔案/目錄 | 說明 | 備份優先度 |
|----------|------|-----------|
| `data/ccstockworkenv.db` | SQLite 研究快取資料庫 | ⭐⭐⭐ 最重要 |
| `config.json` | Telegram Token、Email API 等機密 | ⭐⭐⭐ 最重要 |
| `output/` | 過去產生的 HTML 報告 | ⭐⭐ 選擇性 |
| `~/.claude/settings.json` | Claude Code Shell 修正設定 | ⭐⭐ 重要 |

### 備份指令

```bash
# 在舊電腦上執行
BACKUP_DIR="/c/Users/<帳號>/Desktop/CCStockWorkEnv_backup_$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

cp data/ccstockworkenv.db "$BACKUP_DIR/"
cp config.json "$BACKUP_DIR/"
cp -r output/ "$BACKUP_DIR/output/" 2>/dev/null || true

# 備份 Claude 設定
cp ~/.claude/settings.json "$BACKUP_DIR/claude_settings.json"

echo "Backup saved to: $BACKUP_DIR"
```

### .venv 虛擬環境

**不需要備份 `.venv/`**。所有虛擬環境由 `uv` 依據各子目錄的 `pyproject.toml` 自動重建：

```bash
# 在新電腦 Clone 後，第一次執行任何 uv run 指令時會自動建立
# 例如：
cd tool_scripts/market_data && uv run python fetcher_factory.py quote AAPL --market US
# uv 會自動建立 .venv 並安裝依賴，無需手動 pip install
```

### 路徑差異注意事項

| 情境 | 舊電腦路徑 | 新電腦調整方式 |
|------|-----------|---------------|
| `.claude/launch.json` 中的 `runtimeExecutable` | `C:/Users/yuutachi/...` | 改為新電腦的 uv 路徑 |
| `~/.claude/settings.json` 中的 `PATH` | `/c/Users/yuutachi/...` | 改為新電腦的帳號名 |

---

## 4. 坑點防禦

### 錯誤清單與解決對策

---

#### 🔴 坑 1：Bash 工具完全無法使用

**錯誤訊息**
```
No suitable shell found. Claude CLI requires a Posix shell environment.
```

**原因**：Windows 的 `SHELL` 環境變數指向 `C:\Program Files\Git\cmd\git.exe`（Git 執行檔），而非 `bash.exe`。

**解決方法**：建立 `~/.claude/settings.json`，設定 `SHELL` 指向 bash：
```json
{
  "env": {
    "SHELL": "C:/Program Files/Git/bin/bash.exe"
  }
}
```
然後**完全重啟 Claude Desktop**（系統匣 → Quit → 重新開啟）。

> ⚠️ 修改 settings.json 不會立即生效，必須重啟程序。

---

#### 🔴 坑 2：`uv: command not found`

**錯誤訊息**
```
/usr/bin/bash: line 1: uv: command not found
```

**原因**：uv 安裝在 Windows 路徑下，Git Bash 的 PATH 不包含它。

**解決方法（二擇一）**：

方法 A — 在 `~/.claude/settings.json` 加入 PATH：
```json
{
  "env": {
    "SHELL": "C:/Program Files/Git/bin/bash.exe",
    "PATH": "/c/Users/<帳號>/AppData/Local/uv/bin:/usr/local/bin:/usr/bin:/bin:/mingw64/bin"
  }
}
```

方法 B — 暫時用完整路徑（不需重啟，立即可用）：
```bash
UV="/c/Users/<帳號>/AppData/Local/uv/bin/uv.exe"
"$UV" run python script.py
```

---

#### 🔴 坑 3：DB 缺少 v2 欄位

**錯誤訊息**
```
sqlite3.OperationalError: table financials has no column named quick_ratio
```

**原因**：`db_manager.py --init` 初始化時直接寫入 schema version 2，但 `SCHEMA_SQL` 未包含 v2 欄位（`quick_ratio` 等 12 欄）。`--migrate` 指令看到版本已是 2 就跳過，導致欄位永遠不會被新增。

**解決方法**：手動執行 ALTER TABLE（見 Step 4 的修正腳本）。

**驗證已修正**：
```bash
"$UV" run python -c "
import sqlite3
conn = sqlite3.connect('../../data/ccstockworkenv.db')
cols = [r[1] for r in conn.execute('PRAGMA table_info(financials)').fetchall()]
assert 'quick_ratio' in cols, 'quick_ratio missing!'
print('Schema OK. Total columns:', len(cols))
"
```

---

#### 🟡 坑 4：`start_server.sh` 在 Windows 無法運行

**原因**：腳本使用 `nohup`、`ps -p` 等 Unix 指令，Git Bash 不完整支援。

**解決方法**：改用直接指令啟動：
```bash
nohup "$UV" run python manage.py runserver 0.0.0.0:8800 > ../../data/webserver.log 2>&1 &
```

---

#### 🟡 坑 5：多行 Bash 指令中的變數消失

**錯誤症狀**：把路徑存入變數 `PROJ=...`，換行後 `cd "$PROJ/..."` 出現 `No such file or directory`。

**原因**：Git Bash 在某些情況下不正確展開跨行變數。

**解決方法**：使用 `&&` 單行串連，或使用 Unix 風格路徑（`/c/Users/...` 而非 `C:/Users/...`）：
```bash
# ❌ 不穩定
PROJ="/c/Users/..."
cd "$PROJ/tool_scripts" && uv run python script.py

# ✅ 穩定
cd "/c/Users/.../tool_scripts" && "/c/.../uv.exe" run python script.py
```

---

#### 🟡 坑 6：`launchd` 排程、`ctb` Telegram Bot 無法使用

**原因**：這些功能設計給 macOS。`launchd` 是 macOS 的系統排程器；`ctb` 使用 Bun 編譯，需要特殊設定才能在 Windows 運行。

**解決方法**：在 Windows 上直接使用 Claude Desktop 操作，不透過 Telegram。排程任務若有需求，可改用 Windows 工作排程器（Task Scheduler）。

---

#### 🟡 坑 7：報告 URL 404

**症狀**：`curl http://localhost:8800/reports/<slug>/` 回傳 404。

**原因**：報告目錄命名格式錯誤。Django 的 regex 掃描器要求格式為 `YYYYMMDD_HHMMSS_<type>`，時間戳必須在前。

```
✅ 正確：20260303_234500_single_2330_TW/
❌ 錯誤：single_2330_TW_20260303_234500/
```

---

## 5. 因果驗證清單

完成安裝後，依序執行以下指令確認每個層級正常：

```bash
# ✅ 第一層：Shell 環境
echo "Shell OK: $SHELL"
# 預期：Shell OK: /c/Program Files/Git/bin/bash.exe

# ✅ 第二層：uv 可執行
uv --version  # 或用完整路徑
# 預期：uv 0.x.x

# ✅ 第三層：資料庫
cd tool_scripts/db_ops
uv run python db_manager.py --info
# 預期：Schema version: 2，tables 列表正常

# ✅ 第四層：市場資料 API
cd ../market_data
uv run python fetcher_factory.py quote AAPL --market US
# 預期：JSON 含 price、change、volume

# ✅ 第五層：財務計算
cd ../financial_calc
uv run python ratios.py AAPL --market US
# 預期：JSON 含 pe_ratio、roe、gross_margin 等

# ✅ 第六層：網頁伺服器
curl -s -o /dev/null -w "%{http_code}" http://localhost:8800/
# 預期：200

# ✅ 第七層：報告產生（在 Claude Code 中執行）
# /check_price AAPL
# /health_check 2330 TW
```

---

## 6. 依賴清單

### requirements.txt（彙整所有子模組）

```text
# ===== 市場資料 (market_data) =====
yfinance>=0.2.36
requests>=2.31.0
akshare>=1.14.0

# ===== 台股資料 (根目錄 / financial_calc) =====
twstock>=1.4.0

# ===== 財務計算 (financial_calc) =====
# yfinance 已上方包含

# ===== 報告產生 (report_gen) =====
matplotlib>=3.8.0

# ===== 網頁伺服器 (web_server) =====
django>=5.1,<5.2

# ===== 通訊工具 (send_telegram / send_mail) =====
# requests 已上方包含
```

> ⚠️ **注意**：本專案使用 `uv` 管理環境，每個子目錄有獨立的 `.venv`，**不建議** 用 `pip install -r requirements.txt` 全域安裝。此 `requirements.txt` 僅供人工參考依賴版本。

### environment.yaml（Conda 參考格式）

```yaml
name: ccstockworkenv
channels:
  - conda-forge
  - defaults
dependencies:
  - python=3.12
  - pip
  - pip:
    - yfinance>=0.2.36
    - requests>=2.31.0
    - akshare>=1.14.0
    - twstock>=1.4.0
    - matplotlib>=3.8.0
    - django>=5.1,<5.2
```

> 本專案實際使用 `uv` 而非 conda。此 yaml 僅供 conda 使用者參考。

### 子模組依賴對照表

| 子模組目錄 | 主要依賴 |
|-----------|---------|
| `tool_scripts/market_data/` | yfinance, requests, akshare |
| `tool_scripts/financial_calc/` | yfinance |
| `tool_scripts/report_gen/` | matplotlib, yfinance |
| `tool_scripts/web_server/` | django 5.1 |
| `tool_scripts/send_telegram/` | requests |
| `tool_scripts/send_mail/` | requests |
| `tool_scripts/db_ops/` | 純標準庫（sqlite3） |

---

## 快速移植速查表

```
新電腦移植三步驟：

1. 安裝軟體
   git + python + uv + Claude Desktop

2. Clone & 還原
   git clone https://github.com/yuutachichao/CCStockWorkEnv.git
   cp backup/config.json config.json
   cp backup/ccstockworkenv.db data/

3. 修正 Windows 設定
   建立 ~/.claude/settings.json（設定 SHELL + PATH）
   重啟 Claude Desktop
   執行 DB schema 修正腳本
   啟動 Django 伺服器
```

---

*文件由 CCStockWorkEnv + Claude Code 自動整理 — 2026-03-03*
