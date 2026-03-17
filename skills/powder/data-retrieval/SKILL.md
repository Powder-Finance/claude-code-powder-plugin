---
name: powder-data-retrieval
description: Use when user asks to fetch, view, or re-display extracted data from a previously uploaded financial document. Retrieves holdings by document ID, handles pagination, and formats results.
---

# Powder Data Retrieval

Retrieve and format data from previously uploaded financial documents. Use when you need to access holdings or transactions from a document that's already been processed.

## Quick Start

1. **Check Status** - Verify document is in `done` status
2. **Fetch Page 1** - Run `powder --json data <id>`
3. **Check Count** - See if results are paginated (`count > page * 100`)
4. **Fetch Remaining Pages** - Loop through `--page 2`, `--page 3`, etc. if needed
5. **Format Output** - Present as table with totals, never raw JSON

## When to Use This Skill

- User asks to "fetch data for document X"
- User says "show me what's in document ID 12345"
- User wants to re-display results from a previous upload
- User needs to export or analyze holdings from a known document ID
- Following up after a timeout - document finished processing later

## Data Retrieval Workflow

### Step 0: Prerequisite Check

Before starting, verify the environment:

```bash
# Check powder CLI is available
which powder

# Verify auth token is set
test -n "$POWDER_API_TOKEN" && echo "set" || echo "not set"
```

**Decision Tree:**
- ✅ Both available → Proceed to Step 1
- ❌ `powder` not found → Suggest running `/Powder:setup` to install the CLI
- ❌ Token not set → Suggest running `/Powder:setup` to configure authentication

### Step 1: Verify Document Status

Before fetching data, confirm the document is ready:

```bash
powder --json status 12345
```

**Expected Response:**
```json
{
  "id": 12345,
  "status": "done",
  "portfolio_id": 67890,
  "closed_at": "2024-01-15T17:23:14.582Z"
}
```

**Decision Tree:**
- ✅ Status = `done` → Proceed to Step 2
- ⏳ Status = `pending`, `processing`, or `in_review` → Wait and poll (see [../upload-and-extract/SKILL.md](../upload-and-extract/SKILL.md))
- ❌ Status = `failed`, `error`, `closed` → No data available, show error details

### Step 2: Fetch First Page

Retrieve the first page of results:

```bash
powder --json data 12345
```

**Response Structure:**
```json
{
  "id": 12345,
  "status": "done",
  "data": {
    "portfolio_id": 67890,
    "ownerships": [
      {
        "name": "VANGUARD TARGET RETIREMENT 2045 FUND",
        "ticker": "VTIVX",
        "quantity": 6250.125,
        "statement_asset_value": 245000.50,
        ...
      }
    ]
  },
  "count": 1,
  "page": 1
}
```

**Key Fields:**
- `count` - Total number of holdings across all pages
- `page` - Current page number (default: 1)
- `data.ownerships` - Array of holdings (max 100 per page)
- `data.portfolio_id` - Portfolio this data belongs to

### Step 3: Handle Pagination

If `count > 100`, fetch remaining pages with `--page 2`, `--page 3`, etc. up to `ceil(count / 100)`. Accumulate all ownerships across pages, then format once.

```bash
powder --json data 12345 --page 2
```

### Step 4: Format and Present Data

Apply formatting rules from [../upload-and-extract/reference/output-formatting.md](../upload-and-extract/reference/output-formatting.md).

## Error Handling

| Error | What to Tell the User |
|-------|----------------------|
| 404 Not Found | "Document not found. Verify the upload ID is correct." |
| Still processing | "Still processing. Check back shortly or use `/Powder:status <id> --watch`." |
| Processing failed | Show the error details from the status response. See [error codes reference](../upload-and-extract/reference/error-codes.md). |

If errors persist after re-uploading, suggest contacting support@powderfi.com with the document ID.

## Related

- [../upload-and-extract/SKILL.md](../upload-and-extract/SKILL.md) - Full upload workflow
- [../upload-and-extract/reference/output-formatting.md](../upload-and-extract/reference/output-formatting.md) - Field definitions, formatting rules, and API response shape
- [../upload-and-extract/reference/error-codes.md](../upload-and-extract/reference/error-codes.md) - Error codes
