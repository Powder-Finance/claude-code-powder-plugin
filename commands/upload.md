---
description: Upload a financial statement to Powder, wait for processing, and return a summary of extracted holdings or transactions.
argument-hint: path to PDF, XLSX, PNG, JPG, or JPEG
---

You are uploading a financial statement to Powder for the user.

## Prerequisites Check

Before proceeding, verify:
1. `powder` CLI is installed: run `which powder`
2. `POWDER_API_TOKEN` environment variable is set: run `echo $POWDER_API_TOKEN | head -c 10`

If either is missing, stop and suggest the user run `/Powder:setup` first.

## Upload and Processing Workflow

1. **Parse and Validate Arguments**: Extract the file path from `$ARGUMENTS`
   - Supported formats: PDF, XLSX, PNG, JPG, JPEG
   - Reject if path contains shell metacharacters (`;`, `&`, `|`, `` ` ``, `$`, `\n`) — tell the user to rename the file
   - Validate the file exists before uploading

2. **Upload**: Run `powder --json upload "$FILE_PATH"` (always quote the path)
   - Capture the upload ID from the response
   - Note the status

3. **Watch for Completion**: Run `powder --json status <upload_id> --watch --timeout 600`
   - This polls every few seconds until the upload completes or times out
   - Maximum wait: 10 minutes (600 seconds)

4. **Handle Results**:
   - **If done**: Run `powder --json data <upload_id>` and format the output
   - **If timeout**: Tell the user the upload is still processing, provide the upload ID, and suggest they check later with `/Powder:status <upload_id>`
   - **If failed/error/closed**: Report the error message from the status response. See the error codes reference in the upload-and-extract skill for details.

## Output Formatting Rules

**NEVER** output raw JSON. **ALWAYS** format the data as human-readable tables.

### For Holdings:
- **Summary line**: "Extracted N holdings from [filename] (Upload ID: [id])"
- **Table columns**: Name, Ticker, Account, Quantity, Statement Value
- **Totals row**: Sum of Statement Values
- **Anomaly flags**: If any holding has warnings or anomalies, highlight them below the table

### For Transactions:
- **Summary line**: "Extracted N transactions from [filename] (Upload ID: [id])"
- **Table columns**: Date, Description, Account, Amount, Type
- **Totals**: Sum of amounts by type (deposits, withdrawals, etc.)

### Security Rules:
- **NEVER** echo full account numbers — mask to last 4 digits (e.g., "****1234")
- **NEVER** output full file paths from the user's system — show filename only
- **NEVER** dump raw JSON to the user

## Example Output

```
✓ Upload complete (ID: abc123)

Extracted 42 holdings from statement_2024.pdf

| Name                          | Ticker | Account  | Quantity | Statement Value |
|-------------------------------|--------|----------|----------|-----------------|
| Apple Inc.                    | AAPL   | ****5678 | 100      | $18,500.00      |
| Vanguard Total Stock Market   | VTI    | ****5678 | 250      | $62,750.00      |
| ...                           | ...    | ...      | ...      | ...             |
|-------------------------------|--------|----------|----------|-----------------|
| **Total**                     |        |          |          | **$247,890.00** |

⚠ 2 holdings flagged for review (unusual tickers or missing data)
```

If you encounter any issues, check the error message and suggest next steps.
