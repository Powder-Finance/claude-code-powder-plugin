# Brokerage Statement Upload and Extract

Complete walkthrough of uploading and extracting data from a brokerage statement using the Powder CLI.

## Example: Samsung NetBenefits Statement

This example uses a real Fidelity NetBenefits statement for a Samsung employee 401(k) account.

---

## Step 1: Upload the Statement

```bash
powder --json upload ~/Documents/samsung-netbenefits.pdf
```

**Response:**
```json
{
  "id": 39011,
  "status": "processing",
  "portfolio_id": null
}
```

The upload has been queued for processing. Note the `id: 39011` - we'll use this to check status and retrieve data.

---

## Step 2: Monitor Processing Status

Use `--watch` to poll the status until processing completes:

```bash
powder --json status 39011 --watch --timeout 600
```

**Response (when complete):**
```json
{
  "id": 39011,
  "status": "done",
  "portfolio_id": 42992,
  "closed_at": "2025-03-16T14:23:45Z"
}
```

- `status: "done"` indicates successful processing
- `portfolio_id: 42992` is the assigned portfolio ID
- Processing took ~30 seconds

**What `--watch` does:**
- Polls status every 2-3 seconds
- Exits when terminal status reached (`done`, `failed`, `error`, `deleted`, `closed`)
- Times out after 600 seconds (10 minutes) by default
- Use `--timeout <seconds>` to adjust

---

## Step 3: Retrieve Extracted Data

```bash
powder --json data 39011
```

**Response:**
```json
{
  "id": 39011,
  "status": "done",
  "data": {
    "portfolio_id": 42992,
    "ownerships": [
      {
        "name": "T ROWE PRICE RETIREMENT BLEND 2045 FUND",
        "account_type": "401k",
        "account_name": "Fidelity NetBenefits *XXXXX1",
        "entity": null,
        "ticker": "TRBQX",
        "isin": "US8727975273",
        "cusip": "872797527",
        "statement_asset_value": 343124.17,
        "statement_cost_basis": null,
        "statement_original_cost_basis": null,
        "statement_gain_loss": null,
        "quantity": 8832.025,
        "statement_price": 38.85,
        "eod_valuations": 13.02,
        "eod_gain_loss": null,
        "eod_asset_value": 114992.97,
        "purchase_date": null,
        "currency": "USD",
        "account_allocation": 1.0,
        "portfolio_allocation": 1.0,
        "dividend_yield": 0.016049,
        "expense_ratio": 0.0042,
        "asset_class_level_1": "Multi-Asset Class",
        "asset_class_level_2": "Multi-Asset Class",
        "asset_class_level_3": "Multi-Asset Class",
        "asset_class_level_4": "Multi-Asset Class",
        "total_commitment": null,
        "unfunded_commitment": null,
        "total_contribution": null,
        "total_distribution": null,
        "vintage": null,
        "statement_date": null,
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
| T Rowe Price Retirement Blend 2045 Fund | TRBQX | 8,832.025 | $343,124.17 | 401k |

**Key Fields:**
- **Investment**: T Rowe Price target-date retirement fund
- **Ticker/ISIN**: `TRBQX` / `US8727975273`
- **Quantity**: 8,832.025 shares
- **Statement Price**: $38.85 per share
- **Statement Value**: $343,124.17 (statement date)
- **Current EOD Value**: $114,992.97 (end-of-day pricing)
- **Account**: Fidelity NetBenefits 401(k) ending in ...XX1
- **Asset Class**: Multi-Asset Class (target-date fund)
- **Expense Ratio**: 0.42%
- **Dividend Yield**: 1.60%

**Note the value difference:**
- Statement value: $343,124.17
- EOD (current) value: $114,992.97

This discrepancy suggests the statement is from an older date, and the fund has declined in value since then. Always check `statement_date` when available.

---

## Summary

**Full workflow:**
1. Upload → Get upload ID
2. Monitor → Wait for `status: "done"`
3. Retrieve → Get extracted holdings data
4. Process → Parse JSON for your application

**Processing time:** Typically 15-60 seconds for a standard brokerage statement.

**Next steps:**
- See the data retrieval skill for handling large statements (100+ holdings)
- See [error-recovery.md](error-recovery.md) for handling failures
