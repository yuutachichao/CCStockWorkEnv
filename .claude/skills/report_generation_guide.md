# Report Generation Guide — 手機友善報告產生規範

所有研究報告必須遵循本指南產生。報告的最終呈現平台是手機瀏覽器,透過 CCStockWorkEnv Web Server 提供。

---

## 核心原則

1. **Mobile-first** — 所有設計以 375px 螢幕寬度為基準
2. **直接寫入 output/** — 報告以 HTML 檔案形式儲存到 `output/YYYYMMDD_HHMM_<type>.html`
3. **瀏覽器驗證** — 每份報告產生後,必須用 Playwright 在手機與桌面模式下截圖驗證
4. **驗證通過後才發佈** — 確認手機顯示效果良好後,才發送連結給使用者
5. **附上原始提問** — 當請求來自 ctb（Telegram）使用者時,在報告底部的免責聲明之前,附上使用者的原始提問（Original Prompt）,方便日後回顧報告時理解產生背景

---

## 報告 HTML 結構

### 基本骨架

```html
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>報告標題 — CCStockWorkEnv</title>
<style>
/* === 必須包含的基礎 CSS === */
/* 見下方「必要 CSS」區塊 */
</style>
</head>
<body>

<header class="report-header">
    <h1>報告標題</h1>
    <div class="meta">
        產生時間：YYYY-MM-DD HH:MM<br>
        資料來源：yfinance / twstock / AKShare
    </div>
</header>

<main>
    <!-- 報告內容放這裡 -->
</main>

<!-- 原始提問：來自 ctb 使用者時附上 -->
<div class="original-prompt">
    <div class="prompt-label">📝 Original Prompt</div>
    <div class="prompt-text">使用者的原始提問內容</div>
</div>

<footer class="disclaimer">
    免責聲明：本報告僅供研究參考,不構成投資建議。
</footer>

<script>
/* === 互動功能 JS === */
/* 見下方「JavaScript 功能」區塊 */
</script>

</body>
</html>
```

---

## 必要 CSS

每份報告的 `<style>` 必須包含以下 CSS。不要省略任何部分。

```css
/* ===== Reset & Base ===== */
* { margin: 0; padding: 0; box-sizing: border-box; }

body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI',
                 'PingFang TC', 'Microsoft JhengHei', sans-serif;
    color: #1a1a1a;
    background: #f8f9fa;
    padding: 16px;
    line-height: 1.6;
    max-width: 800px;
    margin: 0 auto;
    -webkit-text-size-adjust: 100%;
}

/* ===== Typography ===== */
h1 {
    font-size: 1.4em;
    color: #2c3e50;
    border-bottom: 3px solid #3498db;
    padding-bottom: 8px;
    margin: 16px 0 12px 0;
    word-wrap: break-word;
}

h2 {
    font-size: 1.2em;
    color: #2c3e50;
    margin: 24px 0 12px 0;
    padding-bottom: 4px;
    border-bottom: 1px solid #eee;
}

h3 {
    font-size: 1.05em;
    color: #34495e;
    margin: 16px 0 8px 0;
}

p { margin: 8px 0; }

/* ===== Header ===== */
.report-header {
    background: linear-gradient(135deg, #2c3e50, #34495e);
    color: white;
    padding: 20px;
    border-radius: 8px;
    margin-bottom: 20px;
}

.report-header h1 {
    color: white;
    border-bottom-color: rgba(255,255,255,0.3);
    margin-top: 0;
}

.meta {
    color: #95a5a6;
    font-size: 0.85em;
    line-height: 1.6;
}

.report-header .meta { color: rgba(255,255,255,0.8); }

/* ===== Tables ===== */
.table-container {
    width: 100%;
    overflow-x: auto;
    -webkit-overflow-scrolling: touch;
    margin: 12px 0;
    border-radius: 6px;
    border: 1px solid #e0e0e0;
}

table {
    border-collapse: collapse;
    width: 100%;
    font-size: 0.85em;
    white-space: nowrap;
}

th {
    background: #2c3e50;
    color: #fff;
    padding: 10px 12px;
    text-align: left;
    font-weight: 600;
    position: sticky;
    top: 0;
}

td {
    padding: 8px 12px;
    border-bottom: 1px solid #eee;
}

tr:nth-child(even) { background: #f8f9fa; }
tr:hover { background: #edf2f7; }

/* First column sticky for wide tables */
.table-sticky-col th:first-child,
.table-sticky-col td:first-child {
    position: sticky;
    left: 0;
    z-index: 1;
    background: inherit;
}

.table-sticky-col th:first-child { z-index: 2; }

/* Scroll hint */
.scroll-hint {
    text-align: center;
    font-size: 0.75em;
    color: #95a5a6;
    padding: 4px 0;
    display: none;
}

/* ===== Cards ===== */
.card {
    background: #fff;
    border: 1px solid #e0e0e0;
    border-radius: 8px;
    padding: 16px;
    margin: 16px 0;
    box-shadow: 0 1px 3px rgba(0,0,0,0.08);
}

.card h3 { margin-top: 0; }

/* ===== Badges ===== */
.badge {
    display: inline-block;
    padding: 3px 10px;
    border-radius: 12px;
    font-size: 0.8em;
    font-weight: 600;
}

.badge-strong { background: #27ae60; color: #fff; }
.badge-pass   { background: #3498db; color: #fff; }
.badge-watch  { background: #f39c12; color: #fff; }
.badge-exclude { background: #e74c3c; color: #fff; }

/* ===== Status Colors ===== */
.text-safe    { color: #27ae60; font-weight: bold; }
.text-grey    { color: #f39c12; font-weight: bold; }
.text-danger  { color: #e74c3c; font-weight: bold; }
.text-info    { color: #3498db; font-weight: bold; }

/* ===== Summary Stats ===== */
.stats-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 12px;
    margin: 16px 0;
}

.stat-card {
    background: white;
    border: 1px solid #e0e0e0;
    border-radius: 8px;
    padding: 16px;
    text-align: center;
}

.stat-card .number {
    font-size: 2em;
    font-weight: 700;
    line-height: 1.2;
}

.stat-card .label {
    font-size: 0.8em;
    color: #666;
    margin-top: 4px;
}

/* ===== Collapsible Sections ===== */
.collapsible {
    cursor: pointer;
    user-select: none;
    display: flex;
    align-items: center;
    justify-content: space-between;
}

.collapsible::after {
    content: '▼';
    font-size: 0.7em;
    transition: transform 0.3s;
    color: #95a5a6;
}

.collapsible.collapsed::after { transform: rotate(-90deg); }
.collapsible-content { overflow: hidden; transition: max-height 0.3s ease; }
.collapsible-content.collapsed { max-height: 0 !important; }

/* ===== Original Prompt ===== */
.original-prompt {
    background: #f0f4f8;
    border: 1px solid #d0d7de;
    border-radius: 8px;
    padding: 16px;
    margin-top: 32px;
}

.original-prompt .prompt-label {
    font-size: 0.8em;
    color: #666;
    font-weight: 600;
    margin-bottom: 8px;
}

.original-prompt .prompt-text {
    font-size: 0.9em;
    color: #2c3e50;
    line-height: 1.6;
    white-space: pre-wrap;
    word-wrap: break-word;
}

/* ===== Footer ===== */
.disclaimer {
    color: #999;
    font-size: 0.8em;
    margin-top: 32px;
    padding: 16px;
    border-top: 1px solid #ddd;
    text-align: center;
}

.footer-gen {
    text-align: center;
    color: #bbb;
    font-size: 0.75em;
    margin-top: 12px;
}

/* ===== Mobile RWD ===== */
@media (max-width: 768px) {
    body { padding: 12px; font-size: 14px; }
    h1 { font-size: 1.3em; }
    h2 { font-size: 1.1em; }
    h3 { font-size: 1em; }

    .report-header { padding: 16px; border-radius: 6px; }
    .report-header h1 { font-size: 1.2em; }

    table { font-size: 0.75em; }
    th, td { padding: 6px 8px; }

    .card { padding: 12px; margin: 12px 0; }
    .stats-grid { grid-template-columns: repeat(2, 1fr); gap: 8px; }
    .stat-card { padding: 12px; }
    .stat-card .number { font-size: 1.5em; }

    .scroll-hint { display: block; }
}

@media (max-width: 380px) {
    body { padding: 8px; font-size: 13px; }
    table { font-size: 0.65em; }
    th, td { padding: 4px 6px; }
    .stats-grid { grid-template-columns: 1fr; }
}

/* ===== Print ===== */
@media print {
    body { background: white; max-width: 100%; }
    .report-header { background: #2c3e50 !important; -webkit-print-color-adjust: exact; }
    .collapsible::after { display: none; }
}
```

---

## JavaScript 功能

每份報告的 `</body>` 前必須包含以下 JS。提供可摺疊區塊和表格滑動提示。

```javascript
document.addEventListener('DOMContentLoaded', function() {
    // 1. Collapsible sections
    document.querySelectorAll('.collapsible').forEach(function(el) {
        el.addEventListener('click', function() {
            this.classList.toggle('collapsed');
            var content = this.nextElementSibling;
            if (content && content.classList.contains('collapsible-content')) {
                content.classList.toggle('collapsed');
                if (!content.classList.contains('collapsed')) {
                    content.style.maxHeight = content.scrollHeight + 'px';
                }
            }
        });
    });

    // 2. Auto-detect tables that need scroll hint
    document.querySelectorAll('.table-container').forEach(function(container) {
        var table = container.querySelector('table');
        if (table && table.scrollWidth > container.clientWidth) {
            var hint = container.previousElementSibling;
            if (hint && hint.classList.contains('scroll-hint')) {
                hint.style.display = 'block';
            }
        }
    });
});
```

---

## HTML 元素使用規範

### 表格 — 必須用 `.table-container` 包裹

```html
<p class="scroll-hint">← 左右滑動查看完整表格 →</p>
<div class="table-container">
    <table>
        <thead>
            <tr><th>代號</th><th>名稱</th><th>現價</th><th>漲跌</th></tr>
        </thead>
        <tbody>
            <tr><td>AAPL</td><td>Apple</td><td>$185</td><td class="text-safe">+2.3%</td></tr>
        </tbody>
    </table>
</div>
```

**規則**:
- 所有 `<table>` 必須被 `<div class="table-container">` 包裹
- 超過 5 欄的表格,加上 `class="table-sticky-col"` 固定第一欄
- 表格前加 `<p class="scroll-hint">` 滑動提示 (手機才顯示)

### 卡片 — 用於單股詳細資訊

```html
<div class="card">
    <h3>000596 — 古井貢酒 <span class="badge badge-strong">STRONG</span></h3>
    <p>
        <b>現價</b>：¥121.21（三年高點 ¥307.07 的 39.5%）<br>
        <b>Z-Score</b>：<span class="text-safe">4.24 (安全區)</span>
    </p>
    <div class="table-container">
        <table>...</table>
    </div>
</div>
```

### 統計摘要 — 用 stats-grid

```html
<div class="stats-grid">
    <div class="stat-card">
        <div class="number text-safe">2</div>
        <div class="label">⭐ 強力機會</div>
    </div>
    <div class="stat-card">
        <div class="number text-info">20</div>
        <div class="label">✅ 合格機會</div>
    </div>
    <div class="stat-card">
        <div class="number text-grey">0</div>
        <div class="label">⚠️ 需觀察</div>
    </div>
    <div class="stat-card">
        <div class="number text-danger">15</div>
        <div class="label">❌ 排除</div>
    </div>
</div>
```

### 可摺疊區塊 — 用於長列表

```html
<h2 class="collapsible">❌ 排除清單（15 檔）</h2>
<div class="collapsible-content">
    <div class="table-container">
        <table>...</table>
    </div>
</div>
```

### 原始提問 — ctb 使用者報告底部附上

當報告由 ctb（Telegram Bot）使用者觸發產生時,在 `</main>` 之後、免責聲明之前,附上使用者的原始提問。這讓日後回顧報告時能理解產生背景。

```html
<div class="original-prompt">
    <div class="prompt-label">📝 Original Prompt</div>
    <div class="prompt-text">使用者在 Telegram 輸入的原始訊息</div>
</div>
```

**規則**:
- 原始提問內容必須完整保留,不做修改或摘要
- 位置固定在 `</main>` 和 `<footer class="disclaimer">` 之間

### 色彩標記 — 條件式著色

```html
<!-- 數值判定 -->
<td class="text-safe">4.24</td>     <!-- Z-Score > 2.99 -->
<td class="text-grey">2.24</td>     <!-- 1.81 < Z-Score < 2.99 -->
<td class="text-danger">0.83</td>   <!-- Z-Score < 1.81 -->

<!-- Badge 標籤 -->
<span class="badge badge-strong">STRONG</span>
<span class="badge badge-pass">PASS</span>
<span class="badge badge-watch">WATCH</span>
<span class="badge badge-exclude">EXCLUDE</span>
```

---

## 色彩調色板

| 用途 | Hex | 類別 |
|------|-----|------|
| 安全/強健/上漲 | `#27ae60` | `.text-safe`, `.badge-strong` |
| 資訊/合格 | `#3498db` | `.text-info`, `.badge-pass` |
| 警告/灰色區 | `#f39c12` | `.text-grey`, `.badge-watch` |
| 危險/排除/下跌 | `#e74c3c` | `.text-danger`, `.badge-exclude` |
| 標題/深色 | `#2c3e50` | header, `th` background |
| 背景 | `#f8f9fa` | body background |
| 輔助文字 | `#95a5a6` | `.meta`, hints |

---

## 報告產生流程

### Step 1: 收集資料（Cache-First）

For each stock in the report:

1. **Check cache**: `research_cache_ops.py --is-fresh <ticker> <market> financials`
2. **If stale/missing**: fetch from API → save to `financials` table (`financial_ops.py --bulk-upsert`) → compute health scores (`financial_ops.py --compute-health`) → mark cache (`research_cache_ops.py --mark`)
3. **If fresh**: read directly from DB (`financial_ops.py --get`, `--get-health`)
4. **Also check metrics cache** (24h freshness) for valuation data

All data used in the report MUST come from the DB after being saved. This ensures consistency and avoids redundant API calls for multi-stock reports.

### Step 2: 產生 HTML 檔案

報告支援兩種存放方式，命名格式相同：**時間戳在前，類型在後**。

#### 單檔報告

寫入 `output/YYYYMMDD_HHMM_<type>.html`。

#### 目錄型報告（多檔或含 metadata）

建立目錄 `output/YYYYMMDD_HHMMSS_<type>/`，內含 `index.html` 和可選的 `metadata.json`。

**⚠️ 命名規則（單檔與目錄皆適用）：時間戳必須在前**

```
✅ 正確: YYYYMMDD_HHMMSS_<type>/       → 20260301_104212_cn_hbm_nand_companies/
❌ 錯誤: <type>_YYYYMMDD_HHMMSS/       → cn_hbm_nand_companies_20260301_104212/
```

- `YYYYMMDD`：日期
- `HHMM` 或 `HHMMSS`：時間（單檔用 4 碼，目錄用 6 碼）
- `<type>`：報告類型 (e.g., `cn_3yr_low_research`, `single_AAPL`, `comparison_AAPL_MSFT`)

**為什麼順序很重要？** Django 報告掃描器 (`report_scanner.py`) 使用 regex `^(\d{8})_(\d{4,6})_(.+)$` 解析目錄名。時間戳不在開頭的目錄會被靜默跳過，導致報告 URL 返回 404。

**URL slug 組成**：slug = `{timestamp}_{type}`，與目錄名一致。
- 目錄 `20260301_104212_cn_hbm_nand_companies/` → slug `20260301_104212_cn_hbm_nand_companies`
- URL: `http://<FIXED_IP>:<EXTERNAL_PORT>/reports/20260301_104212_cn_hbm_nand_companies/`（IP 和 port 從 `config.json` → `web_server.fixed_ip` + `web_server.external_port` 讀取）

**目錄型報告必須包含 metadata.json**（與 index.html 同層）：
```json
{
  "title": "中文報告標題（顯示在首頁）",
  "created_at": "20260301_104212",
  "category": "financial_analysis"
}
```
`title` 欄位**必須是繁體中文**，這是首頁和報告列表顯示的標題。沒有 metadata.json 的報告會顯示英文 slug 作為標題。

**iframe 圖表必須用相對路徑**（NEVER use localhost or absolute URL）：
```html
✅ <iframe src="/charts/TICKER/?market=XX&period=1y&embed=1" ...>
❌ <iframe src="http://localhost:8800/charts/TICKER/..." ...>
```
手機用戶透過外部 IP 存取報告，`localhost` 在手機上無法連線。

**HTML 必須**：
- 包含完整的 `<!DOCTYPE html>` 結構
- 包含 `<meta name="viewport">` tag
- 包含本指南的完整 CSS
- 包含本指南的 JavaScript
- 所有表格用 `.table-container` 包裹
- 所有數值使用正確的色彩類別

### Step 3: 瀏覽器驗證 (必要)

報告產生後,**必須**使用 Playwright 截圖驗證手機與桌面顯示效果。

**驗證步驟**:

```bash
# 1. 確認 web server 運行中
curl -s -o /dev/null -w "%{http_code}" http://localhost:8800 | grep -q 200 || \
  (cd tool_scripts/web_server && bash start_server.sh)

# 2. 確認報告 URL 回傳 HTTP 200
curl -s -o /dev/null -w "%{http_code}" http://localhost:8800/reports/<SLUG>/
# 如果 404 → 目錄命名錯誤,先修正再繼續

# 3. Playwright 手機模式截圖 (375x812)
npx playwright screenshot --browser chromium --viewport-size "375,812" \
  --full-page --wait-for-timeout 3000 \
  "http://localhost:8800/reports/<SLUG>/" /tmp/report_mobile.png

# 4. Playwright 桌面模式截圖 (1280x800)
npx playwright screenshot --browser chromium --viewport-size "1280,800" \
  --full-page --wait-for-timeout 3000 \
  "http://localhost:8800/reports/<SLUG>/" /tmp/report_desktop.png

# 5. 用 Read 工具讀取兩張截圖,逐項檢查下方清單
```

**驗證檢查清單**:
- [ ] 手機模式 (375px): 標題完整顯示,無截斷
- [ ] 手機模式: 表格在 `.table-container` 內,可左右滑動
- [ ] 手機模式: 卡片和 badge 正確顯示
- [ ] 手機模式: 文字可讀 (≥ 13px),不會太小
- [ ] 手機模式: 無水平溢出 (body 無橫向 scrollbar)
- [ ] 手機模式: 色彩標記正確 (綠/橙/紅)
- [ ] 桌面模式 (1280px): 排版正常
- [ ] 所有連結和按鈕可點擊
- [ ] 如有佈局問題,修正 HTML 後重新截圖驗證

### Step 4: 發佈

驗證通過後,發送連結到 Telegram。

```bash
cd tool_scripts/send_telegram && uv run python send_message.py \
  --message "📊 <b>報告已產生</b>

類型: {report_display_name}
時間: {formatted_time}

🔗 <a href='{url}'>點此瀏覽報告</a>

💡 最佳體驗: 點擊連結在手機瀏覽器開啟"
```

---

## 範例：完整報告 HTML

以下是一份符合規範的完整報告範例:

```html
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>個股分析：AAPL — CCStockWorkEnv</title>
<style>
/* 包含上方「必要 CSS」的所有內容 */
</style>
</head>
<body>

<header class="report-header">
    <h1>個股分析：AAPL (Apple Inc.)</h1>
    <div class="meta">
        產生時間：2026-03-01 10:30<br>
        市場：US｜資料來源：yfinance
    </div>
</header>

<main>

<div class="stats-grid">
    <div class="stat-card">
        <div class="number">$185.50</div>
        <div class="label">現價</div>
    </div>
    <div class="stat-card">
        <div class="number text-safe">4.52</div>
        <div class="label">Z-Score (安全區)</div>
    </div>
    <div class="stat-card">
        <div class="number text-safe">8/9</div>
        <div class="label">F-Score</div>
    </div>
    <div class="stat-card">
        <div class="number">28.5</div>
        <div class="label">P/E</div>
    </div>
</div>

<h2>財務指標</h2>

<p class="scroll-hint">← 左右滑動查看完整表格 →</p>
<div class="table-container">
    <table>
        <thead>
            <tr><th>指標</th><th>數值</th><th>判定</th></tr>
        </thead>
        <tbody>
            <tr><td>Z-Score</td><td class="text-safe">4.52</td><td>安全區</td></tr>
            <tr><td>F-Score</td><td><b>8/9</b></td><td>強健</td></tr>
            <tr><td>ROE</td><td>25.3%</td><td>優秀</td></tr>
            <tr><td>P/E</td><td>28.5</td><td>合理</td></tr>
        </tbody>
    </table>
</div>

<h2 class="collapsible">詳細財務數據</h2>
<div class="collapsible-content">
    <p class="scroll-hint">← 左右滑動查看完整表格 →</p>
    <div class="table-container">
        <table class="table-sticky-col">
            <thead>
                <tr><th>年度</th><th>營收</th><th>淨利</th><th>EPS</th><th>ROE</th><th>ROA</th></tr>
            </thead>
            <tbody>
                <tr><td><b>2025</b></td><td>394B</td><td>97B</td><td>6.42</td><td>25.3%</td><td>15.1%</td></tr>
                <tr><td><b>2024</b></td><td>383B</td><td>94B</td><td>6.13</td><td>24.1%</td><td>14.8%</td></tr>
            </tbody>
        </table>
    </div>
</div>

</main>

<!-- 原始提問 (ctb 使用者) -->
<div class="original-prompt">
    <div class="prompt-label">📝 Original Prompt</div>
    <div class="prompt-text">幫我查一下 AAPL 的財務狀況</div>
</div>

<footer class="disclaimer">
    免責聲明：本報告僅供研究參考,不構成投資建議。投資有風險,入市需謹慎。
</footer>
<p class="footer-gen">報告由 CCStockWorkEnv 自動產生 — 2026-03-01 10:30</p>

<script>
document.addEventListener('DOMContentLoaded', function() {
    document.querySelectorAll('.collapsible').forEach(function(el) {
        el.addEventListener('click', function() {
            this.classList.toggle('collapsed');
            var content = this.nextElementSibling;
            if (content && content.classList.contains('collapsible-content')) {
                content.classList.toggle('collapsed');
                if (!content.classList.contains('collapsed')) {
                    content.style.maxHeight = content.scrollHeight + 'px';
                }
            }
        });
    });
    document.querySelectorAll('.table-container').forEach(function(container) {
        var table = container.querySelector('table');
        if (table && table.scrollWidth > container.clientWidth) {
            var hint = container.previousElementSibling;
            if (hint && hint.classList.contains('scroll-hint')) {
                hint.style.display = 'block';
            }
        }
    });
});
</script>

</body>
</html>
```

---

## 禁止事項

1. **不要**產生沒有 `<meta name="viewport">` 的 HTML
2. **不要**使用 `max-width: 800px` 作為唯一寬度控制 (必須有 RWD media queries)
3. **不要**讓 `<table>` 裸露在外面,必須用 `.table-container` 包裹
4. **不要**在手機上使用 `white-space: nowrap` 於非表格文字
5. **不要**使用固定像素寬度 (如 `width: 600px`)
6. **不要**跳過瀏覽器驗證步驟
7. **不要**在驗證失敗時就發送報告連結
8. **不要**使用外部 CDN (報告必須完全自包含)
