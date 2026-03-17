# Powder Error Codes Reference

Complete list of error codes returned by the Powder API and CLI.

## File Validation Errors

### PDF Errors

| Code | When It Occurs | Message | Fix |
|------|----------------|---------|-----|
| `PASSWORD_PROTECTED_PDF` | PDF requires password to open | "PDF file is password-protected. Please remove the password and try again." | Remove password protection using Preview, Adobe Acrobat, or similar tool. Save as new unprotected PDF. |
| `CORRUPT_PDF` | PDF file structure is damaged | "PDF file is corrupted and cannot be processed." | Re-download or re-export the PDF from source. Verify file opens locally. |
| `TRUNCATED_PDF` | PDF file is incomplete | "PDF file appears to be incomplete or truncated." | Re-download the full file. Check download completed successfully. |
| `UNREADABLE_PDF` | PDF content cannot be parsed | "PDF file cannot be read. It may be damaged or in an unsupported format." | Try re-exporting from source application. Convert to different PDF version if possible. |
| `EMPTY_PDF` | PDF has no content | "PDF file is empty or contains no readable content." | Ensure source document has content. Re-export. |
| `INVALID_PDF_HEADER` | File claims to be PDF but isn't | "File does not appear to be a valid PDF." | Verify file is actually a PDF (check with `file` command). Re-save as PDF. |
| `PDF_FILE_READ_ERROR` | System cannot read PDF file | "Error reading PDF file. Please check file permissions and try again." | Check file permissions (`ls -l`). Verify file is not locked or in use. |

### Excel Errors

| Code | When It Occurs | Message | Fix |
|------|----------------|---------|-----|
| `PASSWORD_PROTECTED_EXCEL` | Excel file requires password | "Excel file is password-protected. Please remove the password and try again." | Open in Excel/LibreOffice, remove password protection, save as new file. |
| `CORRUPT_EXCEL_ZIP` | Excel file structure is damaged (Excel files are ZIP archives) | "Excel file is corrupted (invalid ZIP structure)." | Re-download or re-export from source. Check file integrity. |
| `INVALID_EXCEL_FILE` | File is not a valid Excel format | "File is not a valid Excel file." | Verify file extension matches content. Re-save as .xlsx. |
| `INVALID_EXCEL_FORMAT` | Excel format not supported | "Excel file format is not supported." | Save as .xlsx format (not .xls or other legacy formats). |
| `EMPTY_EXCEL` | Excel has no sheets or data | "Excel file is empty or contains no readable data." | Ensure workbook has content. Check sheets are not hidden. |
| `EXCEL_FILE_TOO_LARGE` | Excel file exceeds size limit | "Excel file exceeds maximum size of 50MB." | Reduce file size by removing unnecessary sheets/data or splitting into multiple files. |
| `EXCEL_FILE_READ_ERROR` | System cannot read Excel file | "Error reading Excel file. Please check file permissions and try again." | Check file permissions. Verify file is not locked or in use. |

### General File Errors

| Code | When It Occurs | Message | Fix |
|------|----------------|---------|-----|
| `UNSUPPORTED_FILE_TYPE` | File type not in allowed list | "File type not supported. Supported formats: .pdf, .xlsx, .png, .jpg, .jpeg" | Convert file to supported format. Check file extension is correct. |

## API Errors

### HTTP Status Codes

| Code | Error | When It Occurs | Fix |
|------|-------|----------------|-----|
| `401` | Unauthorized | Missing or invalid API token | Set `POWDER_API_TOKEN` environment variable with valid token. |
| `403` | Forbidden | Token valid but lacks permissions for this resource | Verify token has correct permissions. Contact admin if needed. |
| `404` | Not Found | Upload ID doesn't exist or was deleted | Verify upload ID is correct. Check upload wasn't deleted. |
| `429` | Too Many Requests | Rate limit exceeded | Wait 60 seconds before retrying. Reduce request frequency. |
| `500` | Internal Server Error | Server-side processing error | Wait a few minutes and retry. Contact support if persists. |
| `503` | Service Unavailable | Service temporarily down | Wait and retry. Check status page for maintenance announcements. |

## Status Field Values

These appear in the `status` field of upload responses:

### Terminal Statuses (Final States)

- `done` - Processing completed successfully
- `failed` - Processing failed (check error field for details)
- `error` - System error occurred during processing
- `deleted` - Upload was deleted
- `closed` - Upload was manually closed/cancelled

### Non-Terminal Statuses (In Progress)

- `pending` - Upload is queued for processing
- `processing` - File is being processed
- `in_review` - Processing complete, awaiting manual review

When using `powder --json status <id> --watch`, the command will poll until a terminal status is reached.

## Error Response Format

Error responses include:

```json
{
  "id": 12345,
  "status": "failed",
  "error": {
    "code": "PASSWORD_PROTECTED_PDF",
    "message": "PDF file is password-protected. Please remove the password and try again."
  },
  "portfolio_id": null
}
```

Always check the `error.code` field for programmatic error handling.
