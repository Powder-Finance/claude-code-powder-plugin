# Brokerage Statement Upload and Extract

Complete walkthrough of uploading and extracting data from a brokerage statement using the Powder CLI.

## Example: 401(k) Statement

This example walks through uploading a Fidelity NetBenefits 401(k) statement.

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
        "name": "VANGUARD TARGET RETIREMENT 2045 FUND",
        "account_type": "401k",
        "account_name": "Fidelity NetBenefits ***",
        "entity": null,
        "ticker": "VTIVX",
        "isin": "US9219097683",
        "cusip": "921909768",
        "statement_asset_value": 245000.50,
        "statement_cost_basis": null,
        "quantity": 6250.125,
        "statement_price": 39.20,
        "eod_asset_value": 237500.47,
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
| Vanguard Target Retirement 2045 Fund | VTIVX | 6,250.125 | $245,000.50 | 401k |

**Key Fields:**
- **Investment**: Vanguard target-date retirement fund
- **Ticker/ISIN**: `VTIVX` / `US9219097683`
- **Quantity**: 6,250.125 shares
- **Statement Price**: $39.20 per share
- **Statement Value**: $245,000.50
- **EOD Value**: $237,500.47 (end-of-day pricing)
- **Account**: Fidelity NetBenefits 401(k)
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
