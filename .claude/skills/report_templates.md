# Report Templates — 報告模板

All reports are generated in 繁體中文 and saved to `output/` directory.

## Report Types

### 1. Single Stock Report (`/gen_report single <ticker>`)

Structure:
```
# <公司名稱> (<代號>) — 個股研究報告
> 產生時間 / 市場 / 貨幣

## 公司概要
名稱、產業、交易所、員工數、國家

## 即時行情
現價、漲跌、開盤、最高、最低、成交量

## 關鍵指標
市值、P/E、P/B、ROE、ROA、D/E、各項利潤率、殖利率、52週高低

## 健康評估
Z-Score (判定) + F-Score (判定) + 詳細分項

## 財務趨勢
4年營收、毛利、營業利益、淨利、EPS、現金流表格

## 免責聲明
```

### 2. Comparison Report (`/gen_report comparison <t1> <t2> ...`)

Structure:
```
# 股票比較報告：<T1> vs <T2> vs <T3>

## 比較總覽
| 指標 | T1 | T2 | T3 |
現價、市值、P/E、P/B、ROE、ROA、D/E、利潤率、殖利率、52週高低

## 個股摘要
每檔股票的簡要分析

## 免責聲明
```

### 3. Screening Report (`/gen_report screening <criteria>`)

Structure:
```
# 篩選結果報告

## 篩選條件
條件列表

## 結果摘要
符合條件數量、市場分佈

## 結果列表
排序的股票表格，含關鍵指標

## 免責聲明
```

### 4. Sector Report (`/gen_report sector <market> <sector>`)

Structure:
```
# <市場> <產業> 產業報告

## 產業概況
產業描述、趨勢

## 成分股一覽
產業內股票列表 + 關鍵指標

## 產業平均
平均 P/E、ROE 等

## 免責聲明
```

## Common Elements

### 數字格式化
- 金額 > 1T → `X.XXB`
- 金額 > 1B → `X.XXB`
- 金額 > 1M → `X.XXM`
- 百分比 → `XX.X%`
- 比率 → `X.XX`

### 評等圖示
- ★★★★★ (5 star) = Excellent
- ★★★★☆ (4 star) = Good
- ★★★☆☆ (3 star) = Average
- ★★☆☆☆ (2 star) = Below Average
- ★☆☆☆☆ (1 star) = Poor

### 免責聲明 (Required in every report)
```
本報告僅供研究參考，不構成投資建議。投資有風險，請自行評估後做出決策。
```
