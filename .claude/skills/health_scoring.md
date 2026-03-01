# Health Scoring — Z-Score, F-Score & Ratios

## Altman Z-Score

Predicts bankruptcy probability for manufacturing companies.

### Formula
```
Z = 1.2×X1 + 1.4×X2 + 3.3×X3 + 0.6×X4 + 1.0×X5
```

| Component | Formula | Measures |
|-----------|---------|----------|
| X1 | Working Capital / Total Assets | Liquidity |
| X2 | Retained Earnings / Total Assets | Profitability history |
| X3 | EBIT / Total Assets | Operating efficiency |
| X4 | Market Cap / Total Liabilities | Solvency |
| X5 | Revenue / Total Assets | Asset utilization |

### Zones
| Zone | Z-Score | Interpretation |
|------|---------|----------------|
| Safe | > 2.99 | Low bankruptcy risk |
| Grey | 1.81 - 2.99 | Moderate risk, needs monitoring |
| Distress | < 1.81 | High bankruptcy risk |

### Usage
```bash
cd tool_scripts/financial_calc && uv run python zscore.py AAPL --market US
```

---

## Piotroski F-Score

Assesses financial strength on a 0-9 scale.

### 9 Criteria

**Profitability (4 points):**
1. ROA > 0
2. Operating Cash Flow > 0
3. ROA increasing YoY
4. Operating Cash Flow > Net Income

**Leverage/Liquidity (3 points):**
5. Long-term debt ratio decreasing
6. Current ratio increasing
7. No new shares issued

**Operating Efficiency (2 points):**
8. Gross margin increasing
9. Asset turnover ratio increasing

### Interpretation
| Score | Strength |
|-------|----------|
| 7-9 | Strong (financially healthy) |
| 4-6 | Average |
| 0-3 | Weak (financial distress signals) |

### Usage
```bash
cd tool_scripts/financial_calc && uv run python fscore.py AAPL --market US
```

---

## Key Financial Ratios & Thresholds

| Metric | 5★ | 4★ | 3★ | 2★ | 1★ |
|--------|-----|-----|-----|-----|-----|
| P/E | ≤10 | ≤15 | ≤20 | ≤30 | >30 |
| P/B | ≤1.0 | ≤1.5 | ≤3.0 | ≤5.0 | >5.0 |
| ROE | ≥25% | ≥20% | ≥15% | ≥10% | <10% |
| ROA | ≥15% | ≥10% | ≥5% | ≥2% | <2% |
| D/E | ≤30 | ≤50 | ≤100 | ≤200 | >200 |
| Current Ratio | ≥3.0 | ≥2.0 | ≥1.5 | ≥1.0 | <1.0 |
| Gross Margin | ≥60% | ≥40% | ≥25% | ≥15% | <15% |
| Op Margin | ≥30% | ≥20% | ≥10% | ≥5% | <5% |
| Net Margin | ≥25% | ≥15% | ≥8% | ≥3% | <3% |
| Div Yield | ≥5% | ≥4% | ≥3% | ≥2% | <2% |

---

## Opportunity Score

Weighted composite score (0-100) combining:

| Category | Weight | Components |
|----------|--------|------------|
| Value | 30% | P/E (60%), P/B (40%) |
| Quality | 25% | ROE, ROA, Margins |
| Safety | 20% | Z-Score, D/E, Current Ratio |
| Momentum | 15% | Price vs N-year high, F-Score |
| Income | 10% | Dividend yield |

### Verdicts
| Score | Verdict |
|-------|---------|
| ≥75 | Strong Opportunity |
| 60-74 | Moderate Opportunity |
| 40-59 | Neutral |
| <40 | Caution |
