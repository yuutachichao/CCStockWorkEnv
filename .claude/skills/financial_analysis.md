# Financial Analysis — Methodology & Framework

## 5-Step Analysis Framework

### Step 1: Market Context
- Identify the market (US/TW/CN) and sector
- Check recent macro conditions and sector trends
- Note any market-specific factors (regulations, holidays, trading hours)

### Step 2: Fundamental Screening
- Apply quantitative filters (P/E, ROE, D/E, etc.)
- Identify stocks meeting initial criteria
- Flag outliers for deeper investigation

### Step 3: Financial Health Check (Cache-First)
- **Check cache**: `research_cache_ops.py --is-fresh <ticker> <market> financials`
- If stale: fetch → save to `financials` → compute health scores → mark cache
- If fresh: read from DB (`financial_ops.py --get-health`)
- Health scores include: Z-Score, F-Score, growth rates, cash flow quality
- Calculate key ratios with star ratings
- Identify red flags

### Step 4: Deep Dive Analysis
- Review 3-5 years of financial trends
- Compare with sector peers
- Analyze revenue growth trajectory
- Assess management quality indicators (capital allocation, buybacks)

### Step 5: Report & Conclusion
- Generate structured report with charts
- Provide 繁體中文 summary
- Flag any value trap indicators
- Always include 免責聲明

## Low-Point Detection

A stock is at a "low point" when:
- Current price ≤ 60% of 3-year high
- OR current price ≤ 70% of 52-week high
- AND not caused by fundamental deterioration

### Value Trap Detection
Before flagging a stock as an opportunity at a low point, check for value traps:

1. **Declining Revenue** (3+ consecutive quarters) → Possible structural decline
2. **Negative Free Cash Flow** (2+ years) → Cash burn
3. **Increasing D/E Ratio** with declining revenue → Debt spiral
4. **Industry Disruption** → Sector obsolescence
5. **Management Issues** → Frequent C-suite changes, accounting restatements

### Filtering Criteria for Low-Point Opportunities
```
PASS if ALL of:
  - Z-Score > 1.81 (not in distress zone)
  - F-Score ≥ 4 (not financially weak)
  - Revenue not declining 3+ quarters
  - FCF positive in at least 1 of last 2 years
  - Current ratio > 1.0

STRONG if additionally:
  - Z-Score > 2.99 (safe zone)
  - F-Score ≥ 7 (strong)
  - ROE > 10%
  - D/E < 100
```

## Cross-Market Considerations

### US Market
- Quarterly earnings cycle (Jan/Apr/Jul/Oct)
- SEC filings (10-K annual, 10-Q quarterly)
- S&P 500 / NASDAQ-100 as benchmarks

### Taiwan Market
- Biannual/quarterly reporting
- Monthly revenue disclosure (unique to Taiwan)
- TWSE 加權指數 as benchmark
- Many tech/semiconductor companies
- Dividend season typically Jul-Sep

### China A-Share Market
- Regulatory environment changes frequently
- State-owned enterprises (SOEs) behave differently
- A-share vs H-share dynamics
- 滬深300 as benchmark
- Trading suspended during holidays (Spring Festival, National Day)

## Output Language

All user-facing analysis output MUST be in 繁體中文:
- Financial terms with Chinese labels
- Metric names in both Chinese and English
- Conclusions and recommendations in Chinese
- Charts with Chinese labels where applicable
