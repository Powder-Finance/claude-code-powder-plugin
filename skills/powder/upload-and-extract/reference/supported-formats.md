# Supported File Formats

The Powder CLI accepts the following file formats for upload.

## Format Table

| Format | Extension | Max Size | Typical Use Case | Notes |
|--------|-----------|----------|------------------|-------|
| **PDF** | `.pdf` | 50 MB | Brokerage statements, bank statements, investment reports | Most common format. Must not be password-protected. |
| **Excel** | `.xlsx` | 50 MB | Exported holdings, transaction logs, portfolio spreadsheets | Modern Excel format only (not .xls). Must not be password-protected. |
| **PNG** | `.png` | 50 MB | Screenshots of statements, scanned documents | Lossless image format. Best for text clarity. |
| **JPEG** | `.jpg`, `.jpeg` | 50 MB | Photos of statements, scanned documents | Compressed image format. Ensure text is legible. |

## File Requirements

### Size Limits
- **Maximum file size:** 50 MB per upload
- Files larger than 50 MB will be rejected with `EXCEL_FILE_TOO_LARGE` or similar error
- For large files, consider splitting into multiple uploads or reducing file size

### Format Requirements

#### PDF Files
- Must be valid PDF format (not just renamed text file)
- Must not be password-protected → Use Preview/Acrobat to remove passwords
- Must not be corrupted or truncated
- Should contain readable text or clear scanned images
- Multi-page PDFs are supported

#### Excel Files
- Must be `.xlsx` format (modern Excel)
- Legacy formats (`.xls`, `.xlsb`, etc.) are **not supported**
- Must not be password-protected
- Must not be corrupted
- Can contain multiple sheets (all will be processed)
- Must not exceed 50 MB

#### Image Files (PNG, JPEG)
- Must contain readable document content
- Text should be clear and high-resolution (at least 150 DPI recommended)
- Avoid blurry photos or low-quality scans
- Can be screenshots or scanned documents
- Multiple pages require separate uploads (one image = one page)

### Permissions
- Files must be readable by the current user
- Check with: `ls -l <path>` (should show `r` permission)

## File Type Detection

The Powder CLI validates file types based on:
1. **File extension** (e.g., `.pdf`, `.xlsx`)
2. **File header/magic bytes** (actual file content)

Renaming a `.txt` file to `.pdf` will **not** work - the file must actually be in the correct format.

Verify file type with:
```bash
file ~/Documents/statement.pdf
# Output: statement.pdf: PDF document, version 1.7
```

## Unsupported Formats

The following formats are **NOT** supported:

- **Text files:** `.txt`, `.csv`, `.tsv`
- **Legacy Excel:** `.xls`, `.xlsb`, `.xlsm`
- **Word documents:** `.doc`, `.docx`
- **Images:** `.gif`, `.bmp`, `.tiff` (use .png or .jpg instead)
- **Archives:** `.zip`, `.tar`, `.gz`
- **Other:** `.json`, `.xml`, `.html`

If you have data in an unsupported format:
- **CSV/TSV** → Open in Excel and save as `.xlsx`
- **Legacy Excel (.xls)** → Open in Excel and save as `.xlsx`
- **Word/Text** → Export/Print to PDF
- **Other images** → Convert to PNG or JPEG

## Best Practices

1. **Prefer PDF for official documents:** Use PDF for brokerage statements, official reports
2. **Use Excel for structured data:** If you have holdings in a spreadsheet, upload as `.xlsx`
3. **Check quality before upload:** Open the file locally to ensure it's readable
4. **Remove passwords:** Always unlock password-protected files before upload
5. **Stay under size limit:** Compress or split files over 50 MB
6. **Use high-resolution images:** If scanning/photographing documents, use good lighting and high DPI

## Examples

**Valid uploads:**
```bash
powder --json upload ~/Documents/statement-2024-q4.pdf
powder --json upload ~/Downloads/holdings.xlsx
powder --json upload ~/Pictures/statement-screenshot.png
powder --json upload ~/Scans/document.jpg
```

**Invalid uploads:**
```bash
powder --json upload ~/Documents/data.csv          # ❌ CSV not supported
powder --json upload ~/Documents/old-format.xls    # ❌ Legacy Excel not supported
powder --json upload ~/Documents/locked.pdf        # ❌ Password-protected
powder --json upload ~/Documents/huge-file.pdf     # ❌ 75 MB (over 50 MB limit)
```

## Error Messages

When uploading an unsupported file:

```json
{
  "error": {
    "code": "UNSUPPORTED_FILE_TYPE",
    "message": "File type not supported. Supported formats: .pdf, .xlsx, .png, .jpg, .jpeg"
  }
}
```

See [`error-codes.md`](error-codes.md) for all possible errors.
