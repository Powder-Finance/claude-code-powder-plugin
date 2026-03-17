# Output Formatting & Data Reference

Rules for presenting Powder API data to users. **Never show raw JSON.**

## API Response Shape

`powder --json data <id>` returns:

```json
{
  "id": 12345,
  "status": "done",
  "data": {
    "portfolio_id": 67890,
    "ownerships": [
      {
        "name": "VANGUARD TARGET RETIREMENT 2045 FUND",
        "account_type": "401k",
        "account_name": "Fidelity NetBenefits ***",
        "ticker": "VTIVX",
        "isin": "US9219097683",
        "cusip": "921909768",
        "statement_asset_value": 245000.50,
        "quantity": 6250.125,
        "statement_price": 39.20,
        "eod_asset_value": 237500.47,
        "currency": "USD",
        "investment_type": "Mutual Fund",
        "asset_class_level_1": "Multi-Asset Class",
        "dividend_yield": 0.016049,
        "expense_ratio": 0.0042
      }
    ]
  },
  "count": 1,
  "page": 1
}
```

- `count`: Total holdings across all pages. `page`: Current page (100 per page).
- If `count > 100`, fetch more with `--page 2`, `--page 3`, etc.

## Ownership Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Full security name |
| `ticker` | string \| null | Ticker symbol. Null for private funds — flag as anomaly. |
| `isin` | string \| null | International Securities ID (12 chars) |
| `cusip` | string \| null | CUSIP identifier (9 chars, US/Canada) |
| `quantity` | number | Shares/units held |
| `statement_asset_value` | number | Value on statement date (USD). Frozen at upload. |
| `statement_price` | number \| null | Price per unit on statement date |
| `eod_asset_value` | number \| null | Current end-of-day value (USD). Updates daily. Null for private funds. |
| `account_name` | string | Account name. **Always mask in output** — show institution + `***` only. |
| `account_type` | string \| null | `"401k"`, `"IRA"`, `"Roth IRA"`, `"Individual"`, `"SEP IRA"`, `"403b"` |
| `investment_type` | string \| null | `"Stock"`, `"Mutual Fund"`, `"ETF"`, `"Bond"`, `"Money Market"`, `"Option"` |
| `asset_class_level_1` | string \| null | `"Equity"`, `"Fixed Income"`, `"Multi-Asset Class"`, `"Cash & Cash Equivalents"`, `"Alternative"` |
| `currency` | string | Always `"USD"` currently |
| `dividend_yield` | number \| null | Annual yield as decimal (e.g., `0.016` = 1.6%) |
| `expense_ratio` | number \| null | Annual fee as decimal (e.g., `0.0042` = 0.42%). Funds only. |

A large gap between `statement_asset_value` and `eod_asset_value` usually means the statement is old, not an error. Null fields → show `N/A` or `-` in tables, skip when summing.

## Formatting Rules

Every successful response must include:

1. **Success header**: `✅ Successfully extracted holdings from <filename>` (basename only)
2. **Summary line**: `Found {count} holdings in portfolio {portfolio_id}`
3. **Holdings table** with columns: Name (40 char max), Ticker, Account (masked), Quantity, Statement Value
4. **Totals**: Statement Value sum, EOD Value sum (if available), holdings count
5. **Anomaly flags** (if any)

For 10+ holdings: show top 5 by value, then `... plus N more holdings`, then totals.

## Privacy Rules (Critical)

- Account numbers → institution name + `***` only (never show digits)
- File paths → basename only (never full path)
- Never include: full account numbers, SSNs, addresses, phone numbers, raw JSON

## Anomaly Flags

Show `⚠️ Anomalies Detected:` if any of:
- Valuation gap >20% between statement and EOD value
- Missing ticker
- Multiple account types in one statement
- Zero quantity with non-zero value
- Negative values

## Example Output

```
✅ Successfully extracted holdings from fidelity-statement.pdf

Found 1 holdings in portfolio 67890

Holdings:
┌──────────────────────────────────────┬────────┬──────────────────────────┬───────────┬─────────────────┐
│ Name                                 │ Ticker │ Account                  │ Quantity  │ Statement Value │
├──────────────────────────────────────┼────────┼──────────────────────────┼───────────┼─────────────────┤
│ VANGUARD TARGET RETIREMENT 2045   │ VTIVX  │ Fidelity NetBenefits *** │  6,250.13 │     $343,124.17 │
└──────────────────────────────────────┴────────┴──────────────────────────┴───────────┴─────────────────┘

Totals:
- Statement Value: $343,124.17
- Current EOD Value: $114,992.97
- Holdings: 1

⚠️ Anomalies Detected:
- Large valuation gap (66.5%) between statement and current EOD price — likely an old statement
```
