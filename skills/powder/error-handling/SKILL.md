---
name: powder-error-handling
description: Troubleshooting guide for Powder CLI errors. Use when encountering upload failures, processing errors, authentication issues, or API errors.
---

# Powder Error Handling

This skill helps diagnose and resolve common errors when using the Powder CLI.

## Error Categories

### Authentication Errors

**Symptoms:**
- `401 Unauthorized` response
- `403 Forbidden` response
- "Authentication required" message

**Cause:**
- Missing or invalid `POWDER_API_TOKEN` environment variable
- Expired token
- Insufficient permissions

**Fix:**
1. Check token is set: `echo $POWDER_API_TOKEN`
2. Verify token is valid (check with team/dashboard)
3. Set token: `export POWDER_API_TOKEN=your_token_here`
4. Retry operation

---

### File Errors

**Symptoms:**
- `UNSUPPORTED_FILE_TYPE` error
- `EMPTY_PDF` or `EMPTY_EXCEL` error
- File size rejection
- Permission denied

**Cause:**
- Unsupported file format (only .pdf, .xlsx, .png, .jpg, .jpeg supported)
- Empty or corrupted file
- File exceeds 50MB limit
- Insufficient read permissions

**Fix:**
1. Verify file format: `file <path>` (must be PDF, Excel, or image)
2. Check file size: `ls -lh <path>` (must be ≤ 50MB)
3. Check permissions: `ls -l <path>` (must be readable)
4. For large files: compress or split if possible
5. For images: ensure they're clear, readable document scans

---

### Processing Errors

**Symptoms:**
- `PASSWORD_PROTECTED_PDF` or `PASSWORD_PROTECTED_EXCEL`
- `CORRUPT_PDF`, `CORRUPT_EXCEL_ZIP`
- `UNREADABLE_PDF`, `INVALID_PDF_HEADER`
- `TRUNCATED_PDF`

**Cause:**
- Document is password-protected
- File is corrupted or incomplete
- PDF structure is invalid
- Excel file has invalid format

**Fix:**
1. **Password-protected**: Remove password protection before upload
   - PDF: Use Preview/Adobe Acrobat to save without password
   - Excel: Open in Excel, save as new file without password
2. **Corrupt/Truncated**: Re-download or re-export the document
3. **Invalid format**: Try exporting to a different format (e.g., PDF → Excel or vice versa)
4. **Unreadable**: Ensure file is not damaged; try opening locally first

---

### API Errors

**Symptoms:**
- `429 Too Many Requests`
- `500 Internal Server Error`
- Connection timeout
- Network errors

**Cause:**
- Rate limiting (too many requests)
- Server-side issues
- Network connectivity problems
- Request timeout (default: 600s for `--watch`)

**Fix:**
1. **Rate limit (429)**: Wait 60 seconds, then retry
2. **Server error (500)**: Wait a few minutes, check status page, retry
3. **Timeout**: Increase timeout with `--timeout <seconds>` (e.g., `--timeout 900`)
4. **Connection error**: Check internet connection, retry

---

## Full Error Code Reference

For a complete list of all validation error codes with detailed explanations, see:
[`reference/error-codes.md`](reference/error-codes.md)

---

## General Troubleshooting Steps

1. **Check the basics**: File exists, correct path, readable permissions
2. **Validate file**: Open locally to ensure it's not corrupted
3. **Check authentication**: `POWDER_API_TOKEN` is set and valid
4. **Read error message**: Most errors include actionable guidance
5. **Check file size/format**: Must be ≤ 50MB and supported type
6. **Retry with `--watch`**: Use `powder --json status <id> --watch --timeout 600` for real-time updates
7. **Contact support**: If issue persists, provide upload ID and error details
