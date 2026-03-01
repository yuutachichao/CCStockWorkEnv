# Database Patterns — SQLite Usage Guide

## Schema Overview

CCStockWorkEnv uses SQLite with WAL mode. Database at `data/ccstockworkenv.db`.

### Tables

| Table | PK | Purpose |
|-------|-----|---------|
| `stocks` | `(ticker, market)` | Stock universe |
| `daily_prices` | `(ticker, market, date)` | OHLCV data |
| `financials` | `(ticker, market, period, period_date)` | Financial statements |
| `screening_results` | `id` (auto) | Saved screening runs |
| `watchlist` | `(ticker, market)` | Tracked stocks |
| `health_scores` | `(ticker, market, score_date)` | Cached scores |
| `schema_version` | `version` | Migration tracking |

### Key Indexes

- `idx_daily_prices_market` on `daily_prices(market)`
- `idx_daily_prices_date` on `daily_prices(date)`
- `idx_stocks_market` on `stocks(market)`
- `idx_stocks_sector` on `stocks(sector)`

## Common Query Patterns

### Get latest price for a stock
```sql
SELECT * FROM daily_prices
WHERE ticker = ? AND market = ?
ORDER BY date DESC LIMIT 1;
```

### Get N-year high/low
```sql
SELECT MIN(low) as period_low, MAX(high) as period_high
FROM daily_prices
WHERE ticker = ? AND market = ?
  AND date >= date('now', '-3 years');
```

### Stocks near N-year lows (≤60% of period high)
```sql
SELECT dp.ticker, dp.market, s.name,
       latest.close as current_price,
       stats.period_high as three_year_high,
       ROUND(latest.close / stats.period_high * 100, 1) as pct_of_high
FROM stocks s
JOIN (
    SELECT ticker, market, close,
           ROW_NUMBER() OVER (PARTITION BY ticker, market ORDER BY date DESC) as rn
    FROM daily_prices
) latest ON s.ticker = latest.ticker AND s.market = latest.market AND latest.rn = 1
JOIN (
    SELECT ticker, market, MAX(high) as period_high
    FROM daily_prices
    WHERE date >= date('now', '-3 years')
    GROUP BY ticker, market
) stats ON s.ticker = stats.ticker AND s.market = stats.market
WHERE latest.close <= stats.period_high * 0.6
  AND s.is_active = 1
ORDER BY pct_of_high;
```

### Get stocks with financial data
```sql
SELECT s.ticker, s.market, s.name, f.pe_ratio, f.roe, f.de_ratio
FROM stocks s
JOIN financials f ON s.ticker = f.ticker AND s.market = f.market
WHERE f.period = 'annual'
  AND f.period_date = (
      SELECT MAX(period_date) FROM financials
      WHERE ticker = f.ticker AND market = f.market AND period = 'annual'
  )
ORDER BY f.roe DESC;
```

## Upsert Pattern

All write operations use `INSERT ... ON CONFLICT ... DO UPDATE`:
```sql
INSERT INTO daily_prices (ticker, market, date, open, high, low, close, volume)
VALUES (?, ?, ?, ?, ?, ?, ?, ?)
ON CONFLICT(ticker, market, date) DO UPDATE SET
    open = COALESCE(excluded.open, daily_prices.open),
    high = COALESCE(excluded.high, daily_prices.high),
    ...
```

## WAL Mode

WAL (Write-Ahead Logging) is enabled on every connection:
```python
conn.execute("PRAGMA journal_mode=WAL")
conn.execute("PRAGMA foreign_keys=ON")
```

Benefits: concurrent reads during writes, better crash recovery.

## Python Usage

```python
from db_manager import get_connection, DB_PATH

conn = get_connection()
rows = conn.execute("SELECT * FROM stocks WHERE market = ?", ("US",)).fetchall()
results = [dict(r) for r in rows]  # sqlite3.Row → dict
conn.close()
```

## Schema Migrations

Managed via `schema_version` table. Add new migrations in `db_manager.py`:
```python
if current < 2:
    conn.execute("ALTER TABLE stocks ADD COLUMN new_field TEXT")
    conn.execute("INSERT INTO schema_version (version) VALUES (2)")
```
