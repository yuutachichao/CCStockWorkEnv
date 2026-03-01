# Market Data — Fetcher Architecture & Usage

## Architecture

```
fetcher_base.py  →  MarketDataFetcher (ABC)
                     ├── get_quote(ticker) → StockQuote
                     ├── get_price_history(ticker, start, end) → list[PriceRecord]
                     ├── get_company_info(ticker) → CompanyInfo
                     ├── get_financials(ticker, period) → list[dict]
                     ├── get_key_metrics(ticker) → dict
                     └── list_tickers(sector) → list[dict]

fetcher_us.py    →  USFetcher    [yfinance]     — free, no key
fetcher_tw.py    →  TWFetcher    [twstock+yf]   — free, no key
fetcher_cn.py    →  CNFetcher    [AKShare+yf]   — free, no key
fetcher_factory.py → get_fetcher(market) + detect_market(ticker)
```

## Market Codes

| Code | Market | Ticker Format | Currency | API |
|------|--------|--------------|----------|-----|
| `US` | US (NYSE/NASDAQ) | Alpha (AAPL, TSLA) | USD | yfinance |
| `TW` | Taiwan (TWSE/TPEx) | Numeric 4-digit (2330) | TWD | twstock + yfinance |
| `CN` | China A-share (SSE/SZSE) | Numeric 6-digit (600519) | CNY | AKShare + yfinance |

## Ticker Auto-Detection

```python
from fetcher_factory import detect_market

detect_market("AAPL")    # → "US"
detect_market("2330")    # → "TW"
detect_market("600519")  # → "CN"
detect_market("000858")  # → "CN"
```

Rules:
- Pure digits, 4 digits → TW
- Pure digits, 6 digits starting with 6/0/3 → CN
- Otherwise → US

## CLI Usage

```bash
cd tool_scripts/market_data

# Get real-time quote
uv run python fetcher_factory.py quote AAPL --market US

# Get company info
uv run python fetcher_factory.py info 2330 --market TW

# Get key metrics
uv run python fetcher_factory.py metrics TSLA

# Get financials
uv run python fetcher_factory.py financials AAPL --market US --period quarterly

# Get price history
uv run python fetcher_factory.py history AAPL --start 2025-01-01 --end 2025-12-31

# List tickers
uv run python fetcher_factory.py list-tickers --market TW
```

## Python Usage

```python
from fetcher_factory import get_fetcher

fetcher = get_fetcher("US")
quote = fetcher.get_quote("AAPL")
print(f"{quote.name}: ${quote.price} ({quote.change_pct:+.2f}%)")
```

## Data Classes

### StockQuote
```python
@dataclass
class StockQuote:
    ticker: str
    market: str
    name: str
    price: float
    change: float
    change_pct: float
    volume: int
    currency: str
    timestamp: str
```

### CompanyInfo
```python
@dataclass
class CompanyInfo:
    ticker: str
    market: str
    name: str
    sector: str | None
    industry: str | None
    ...
```

## Rate Limits & Best Practices

- **yfinance**: No official rate limit, but avoid >2000 requests/hour
- **twstock**: Real-time data available during TWSE trading hours (09:00-13:30 TST)
- **AKShare**: No rate limit documented, be respectful with batch requests
- Always add 0.1-0.5s delay between requests in batch operations
- Cache results in SQLite to avoid redundant API calls

## Error Handling

Fetchers let errors propagate (fail-fast). Common errors:
- `KeyError` on `info["currentPrice"]` → Stock may be delisted or ticker wrong
- `requests.exceptions.ConnectionError` → Network issue
- Empty DataFrame → No data available for the period

## Extending with New Markets

1. Create `fetcher_xx.py` implementing `MarketDataFetcher`
2. Add to `_FETCHERS` dict in `fetcher_factory.py`
3. Update `detect_market()` rules if needed
