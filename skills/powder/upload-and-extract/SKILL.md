---
name: powder-upload-and-extract
description: Use when user asks to upload, analyze, or extract data from a financial statement (PDF, Excel, or image). Handles brokerage statements, 401k statements, and account documents. Validates files, monitors processing, and formats results as human-readable tables.
---

# Powder Upload and Extract

Automates the complete workflow of uploading financial statements (brokerage, bank, credit card) to the Powder API and extracting structured data. Handles validation, status monitoring, error recovery, and formatting of results.

## Quick Start

1. **Verify Prerequisites** - Check `powder` CLI is installed and `POWDER_API_TOKEN` is set
2. **Validate File** - Ensure file exists, is supported type (.pdf/.xlsx/.png/.jpg/.jpeg), and under 50MB
3. **Upload** - Run `powder --json upload <path> [--portfolio <id>]`
4. **Monitor** - Watch processing with `powder --json status <id> --watch --timeout 600`
5. **Extract & Format** - On success, fetch data and present as formatted table with totals and anomaly flags

## When to Use This Skill

- User asks to "analyze this statement", "extract holdings", "upload brokerage statement"
- User provides a file path to a financial document
- User wants to know what's in a PDF/Excel statement
- User needs portfolio composition or transaction history

## Upload and Extract Workflow

### Step 0: Prerequisite Check

Before starting, verify the environment:

```bash
# Check powder CLI is available
which powder

# Verify auth token is set
echo $POWDER_API_TOKEN | head -c 10
```

**Decision Tree:**
- ✅ Both available → Proceed to Step 1
- ❌ `powder` not found → Suggest running `/Powder:setup` to install the CLI
- ❌ Token not set → Suggest running `/Powder:setup` to configure authentication

### Step 1: Validate File

Before uploading, check the file meets requirements:

```bash
# Check file exists and get size
ls -lh "$FILE_PATH"

# Verify extension
file "$FILE_PATH"
```

**Validation Checklist:**
- [ ] File exists and is readable
- [ ] Extension is one of: `.pdf`, `.xlsx`, `.png`, `.jpg`, `.jpeg`
- [ ] Size is under 50MB (52,428,800 bytes)
- [ ] File is not empty (> 0 bytes)

**Decision Tree:**
- ✅ All checks pass → Proceed to Step 2
- ❌ File not found → Ask user to provide correct path
- ❌ Unsupported extension → Tell the user which formats are supported. Common unsupported types: `.csv` and `.xls` (convert to `.xlsx`), `.doc`/`.docx` (export to PDF), `.html` (save as PDF)
- ❌ Too large → Ask user to split or compress the file
- ❌ Empty file → File is corrupted or incomplete

### Step 2: Upload

Execute the upload command with JSON output:

```bash
powder --json upload "$FILE_PATH"
```

**Optional Flags:**
- `--type brokerage` - Statement type (default, currently the only supported type — can be omitted)
- `--portfolio <id>` - Associate with existing portfolio ID

**Expected Response:**
```json
{
  "id": 39011,
  "status": "processing",
  "portfolio_id": 42992
}
```

**Capture:**
- `id` - Document ID for status tracking
- `portfolio_id` - Portfolio this data belongs to

**Error Handling:**
- 401 Unauthorized → Check `POWDER_API_TOKEN` is valid
- 413 Payload Too Large → File exceeds 50MB limit
- 400 Bad Request → Check file extension and type

### Step 3: Watch Processing Status

Monitor the document processing with automatic polling:

```bash
powder --json status 39011 --watch --timeout 600
```

**Flags:**
- `--watch` - Poll every 2 seconds until terminal status
- `--timeout 600` - Wait up to 10 minutes (recommended for complex statements)

**Terminal Statuses:**
- `done` → Success, proceed to Step 4
- `failed` → Extraction failed, see error details
- `error` → Processing error, see error details
- `closed` → Document was closed or cancelled

**Non-Terminal Statuses:**
- `pending` → Upload is queued for processing
- `processing` → OCR/extraction in progress
- `in_review` → Manual review required (rare)

**Decision Tree:**
- ✅ Status = `done` → Proceed to Step 5 (fetch data)
- ⏱️ Timeout reached → Document still processing, suggest: `powder status <id>` to check later
- ❌ Status = `failed`, `error`, or `closed` → Go to Step 4 (error recovery)

### Step 4: Error Recovery

When processing fails, extract error details:

```bash
powder --json status 39011
```

**Common Error Scenarios:**

| Error Code | Meaning | Fix |
|------------|---------|-----|
| `PASSWORD_PROTECTED_PDF` | PDF requires password | Remove password protection and re-upload |
| `CORRUPT_PDF` / `INVALID_PDF_HEADER` | File is damaged | Re-download or re-export the PDF |
| `UNSUPPORTED_FILE_TYPE` | Wrong file format | Convert to supported format (.pdf, .xlsx, .png, .jpg) |
| `EMPTY_PDF` / `EMPTY_EXCEL` | No extractable content | Check source file has visible content |

See [reference/error-codes.md](reference/error-codes.md) for the full error code reference.

**Next Steps:**
- If fixable → Tell the user what's wrong and suggest they fix the source file and re-upload
- If unfixable or unclear → Report the error details and suggest contacting support@powderfi.com with the document ID

### Step 5: Fetch and Format Data

Once status is `done`, retrieve the extracted data:

```bash
powder --json data 39011
```

**Response Structure:**
```json
{
  "id": 39011,
  "status": "done",
  "data": {
    "portfolio_id": 42992,
    "ownerships": [...]
  },
  "count": 1,
  "page": 1
}
```

**Formatting Rules** (see [reference/output-formatting.md](reference/output-formatting.md)):

1. **Summary Line**: `Found {count} holdings in portfolio {portfolio_id}`
2. **Holdings Table**: Name, Ticker, Account, Quantity, Statement Value
3. **Totals Row**: Sum of Statement Value, EOD Value, count
4. **Anomaly Flags**: Highlight >20% gaps, missing tickers, multiple accounts
5. **Never show**: Raw JSON, unmasked account numbers, full file paths

**Example Output:**

```
Found 1 holdings in portfolio 42992

Holdings:
┌──────────────────────────────────────┬────────┬──────────────────────────┬───────────┬─────────────────┐
│ Name                                 │ Ticker │ Account                  │ Quantity  │ Statement Value │
├──────────────────────────────────────┼────────┼──────────────────────────┼───────────┼─────────────────┤
│ T ROWE PRICE RETIREMENT BLEND 2045   │ TRBQX  │ Fidelity NetBenefits *** │  8,832.03 │     $343,124.17 │
└──────────────────────────────────────┴────────┴──────────────────────────┴───────────┴─────────────────┘

Totals:
- Statement Value: $343,124.17
- Current EOD Value: $114,992.97
- Holdings: 1

⚠️ Anomalies Detected:
- Large valuation gap (66.5% difference) between statement and current EOD price
```

## Examples

See [reference/output-formatting.md](reference/output-formatting.md) for the full API response shape, field definitions, and formatted output example.

## Related Skills

- [../data-retrieval/SKILL.md](../data-retrieval/SKILL.md) - Retrieve previously uploaded data

## Reference Documentation

- [reference/output-formatting.md](reference/output-formatting.md) - Formatting rules and privacy requirements
- [reference/error-codes.md](reference/error-codes.md) - All error codes with user-facing messages
