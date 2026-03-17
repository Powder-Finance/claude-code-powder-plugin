# Brokerage Statement Upload and Extract

Complete walkthrough of uploading and extracting data from a brokerage statement using the Powder CLI.

## Example: 401(k) Statement

This example walks through uploading a 401(k) brokerage statement.

---

## Step 1: Upload the Statement

```bash
powder --json upload ~/Documents/401k-statement.pdf
```

**Response:**
```json
{
  "id": 12345,
  "status": "processing",
  "portfolio_id": null
}
```

The upload has been queued for processing. Note the `id: 12345` — we'll use this to check status and retrieve data.

---

## Step 2: Monitor Processing Status

Use `--watch` to poll the status until processing completes:

```bash
powder --json status 12345 --watch --timeout 600
```

**Response (when complete):**
```json
{
  "id": 12345,
  "status": "done",
  "portfolio_id": 67890
}
```

- `status: "done"` indicates successful processing
- `portfolio_id: 67890` is the assigned portfolio ID

**What `--watch` does:**
- Polls status every 2-3 seconds
- Exits when terminal status reached (`done`, `failed`, `error`, `deleted`, `closed`)
- Times out after 600 seconds (10 minutes) by default
- Use `--timeout <seconds>` to adjust

---

## Step 3: Retrieve Extracted Data

```bash
powder --json data 12345
```

**Response:**
```json
{
  "id": 12345,
  "status": "done",
  "data": {
    "portfolio_id": 67890,
    "ownerships": [
      {
        "name": "EXAMPLE TARGET DATE 2045 FUND",
        "account_type": "401k",
        "account_name": "Brokerage Account ***",
        "entity": null,
        "ticker": "EXMPL",
        "isin": "US0000000000",
        "cusip": "000000000",
        "statement_asset_value": 50000.00,
        "statement_cost_basis": null,
        "quantity": 1000.000,
        "statement_price": 50.00,
        "eod_asset_value": 48500.00,
        "currency": "USD",
        "account_allocation": 1.0,
        "portfolio_allocation": 1.0,
        "dividend_yield": 0.0185,
        "expense_ratio": 0.0008,
        "asset_class_level_1": "Multi-Asset Class",
        "investment_type": "Mutual Fund",
        "statement_page_number": 2
      }
    ]
  },
  "count": 1,
  "page": 1
}
```

---

## Step 4: Format and Display Results

The extracted data shows:

| Holding | Ticker | Shares | Statement Value | Account Type |
|---------|--------|--------|-----------------|--------------|
| Example Target Date 2045 Fund | EXMPL | 1,000.000 | $50,000.00 | 401k |

**Key Fields:**
- **Investment**: Target-date retirement fund
- **Ticker/ISIN**: `EXMPL` / `US0000000000`
- **Quantity**: 1,000.000 shares
- **Statement Price**: $50.00 per share
- **Statement Value**: $50,000.00
- **EOD Value**: $48,500.00 (end-of-day pricing)
- **Account**: Brokerage 401(k)
- **Asset Class**: Multi-Asset Class (target-date fund)
- **Expense Ratio**: 0.08%
- **Dividend Yield**: 1.85%

---

## Summary

**Full workflow:**
1. Upload → Get upload ID
2. Monitor → Wait for `status: "done"`
3. Retrieve → Get extracted holdings data
4. Format → Present as table with totals

**Processing time:** Typically 15-60 seconds for a standard brokerage statement.

**Next steps:**
- See the data retrieval skill for handling large statements (100+ holdings)
- See [error-recovery.md](error-recovery.md) for handling failures
