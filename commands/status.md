---
description: Check the processing status of a Powder upload. Optionally watch until complete.
argument-hint: upload ID
---

You are checking the status of a Powder upload for the user.

## Prerequisites Check

Before proceeding, verify:
1. `powder` CLI is installed: run `which powder`
2. API token is available: run `test -n "${POWDER_API_TOKEN:-$CLAUDE_PLUGIN_OPTION_POWDER_API_TOKEN}" && echo "set" || echo "not set"`

If either is missing, stop and suggest the user run `/Powder:setup` first.

## Important: Token Passthrough

Always prefix `powder` commands with the token bridge so the CLI picks up the plugin-configured token:

```bash
POWDER_API_TOKEN="${POWDER_API_TOKEN:-$CLAUDE_PLUGIN_OPTION_POWDER_API_TOKEN}" powder ...
```

## Status Check Workflow

1. **Parse and Validate Arguments**: Extract the upload ID and any flags from `$ARGUMENTS`
   - Upload ID must be numeric only — reject anything containing non-digit characters and tell the user "Upload IDs are numbers (e.g. 12345)"
   - Optional: `--watch` flag to poll until complete
   - Optional: `--timeout N` to set watch timeout (default 600 seconds)

2. **Run Status Check**: Run `powder --json status "$UPLOAD_ID"` (plus any validated flags)
   - Parse the JSON response
   - Extract: status, filename, created_at, updated_at, error messages (if any)

3. **Interpret Status**: Translate the status code into plain English

## Status Interpretation

Map the status field to user-friendly messages:

- **`pending`**: "Upload is queued for processing"
- **`processing`**: "Currently extracting data from the file"
- **`in_review`**: "Extraction complete, undergoing quality review"
- **`done`**: "✓ Processing complete and ready"
- **`failed`**: "✗ Processing failed"
- **`error`**: "✗ An error occurred"
- **`closed`**: "Upload was closed or cancelled"
- **`deleted`**: "Upload was deleted"

## Response Actions

### If status is `done`:
- Congratulate the user: "✓ Upload [id] is complete!"
- Suggest next step: "Run `/Powder:data <upload_id>` to view the extracted data."

### If status is `processing` or `in_review`:
- Tell the user the current state
- Offer to watch: "Would you like me to watch this upload until it completes? I can add the `--watch` flag."
- If `--watch` is already present, the command will poll automatically

### If status is `failed`, `error`, or `closed`:
- Report the status clearly: "✗ Upload [id] failed"
- If there's an error message in the response, show it (sanitized — no stack traces)
- Suggest troubleshooting:
  - "Check the file format (PDF, XLSX, PNG, JPG, JPEG supported)"
  - "Ensure the file is a valid financial statement"
  - "Contact support@powderfi.com if the issue persists"

## Output Format

Provide a clean, human-readable status summary:

```
Upload 12345
Status: Processing
File: statement_2024.pdf
Uploaded: 2 minutes ago
Last updated: 30 seconds ago

Currently extracting data from the file...
⏳ This usually takes 1-3 minutes.

Would you like me to watch this upload until it completes?
```

Or for a completed upload:

```
✓ Upload 12345 is complete!
File: statement_2024.pdf
Uploaded: 5 minutes ago
Completed: 2 minutes ago

Run `/Powder:data 12345` to view the extracted data.
```

Or for a failed upload:

```
✗ Upload 12345 failed
File: statement_2024.pdf
Error: Unsupported file format

Troubleshooting:
- Ensure the file is PDF, XLSX, PNG, JPG, or JPEG
- Check that the file is a valid financial statement
- Contact support@powderfi.com if the issue persists
```

**Never** dump raw JSON status responses to the user.
