---
description: Fetch extracted holdings or transactions from a processed upload.
argument-hint: upload ID
---

You are fetching extracted data from a Powder upload for the user.

## Prerequisites Check

Before proceeding, verify:
1. `powder` CLI is installed: run `which powder`
2. `POWDER_API_TOKEN` environment variable is set: run `echo $POWDER_API_TOKEN | head -c 10`

If either is missing, stop and suggest the user run `/Powder:setup` first.

## Data Retrieval Workflow

1. **Parse and Validate Arguments**: Extract the upload ID from `$ARGUMENTS`
   - Upload ID must be numeric only — reject anything containing non-digit characters and tell the user "Upload IDs are numbers (e.g. 39011)"

2. **Check Status First**: Run `powder --json status "$UPLOAD_ID"`
   - **If done**: Proceed to fetch data
   - **If processing/in_review**: Tell the user the upload is still processing and offer to watch it: "Would you like me to watch this upload until it completes? I can watch it with `/Powder:status <id> --watch`, or you can check back later."
   - **If failed/closed**: Report the error and stop

3. **Fetch Data**: Run `powder --json data "$UPLOAD_ID"`
   - Parse the JSON response
   - Check for pagination (if `count > 100`, there may be more pages)

4. **Handle Pagination**:
   - If the response contains more than 100 items, inform the user: "This upload contains N items. Showing page 1 of M. To see page 2, use: `/Powder:data <upload_id> --page 2`"
   - If a page number was provided, append `--page N` to the command

## Output Formatting Rules

**NEVER** output raw JSON. **ALWAYS** format the data as human-readable tables.

### For Holdings:
- **Summary line**: "N holdings from Upload [id]"
- **Table columns**: Name, Ticker, Account, Quantity, Statement Value
- **Totals row**: Sum of Statement Values
- **Anomaly flags**: If any holding has warnings or anomalies, highlight them below the table

### For Transactions:
- **Summary line**: "N transactions from Upload [id]"
- **Table columns**: Date, Description, Account, Amount, Type
- **Totals**: Sum of amounts by type (deposits, withdrawals, etc.)

### Security Rules:
- **NEVER** echo full account numbers — mask to last 4 digits (e.g., "****1234")
- **NEVER** dump raw JSON to the user

## Example Output

```
42 holdings from Upload abc123

| Name                          | Ticker | Account  | Quantity | Statement Value |
|-------------------------------|--------|----------|----------|-----------------|
| Apple Inc.                    | AAPL   | ****5678 | 100      | $18,500.00      |
| Vanguard Total Stock Market   | VTI    | ****5678 | 250      | $62,750.00      |
| ...                           | ...    | ...      | ...      | ...             |
|-------------------------------|--------|----------|----------|-----------------|
| **Total**                     |        |          |          | **$247,890.00** |

⚠ 2 holdings flagged for review
```

If the upload is still processing, offer to watch it or check back later.
