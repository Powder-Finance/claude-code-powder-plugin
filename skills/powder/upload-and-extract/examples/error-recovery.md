# Error Recovery Examples

Common error scenarios and how to recover from them.

---

## Scenario 1: Password-Protected PDF

### What Happened

Attempted to upload a brokerage statement PDF that requires a password to open.

### Error Output

**Step 1: Upload**
```bash
powder --json upload ~/Documents/statement-q4-2024.pdf
```

```json
{
  "id": 40123,
  "status": "processing",
  "portfolio_id": null
}
```

**Step 2: Check Status**
```bash
powder --json status 40123 --watch
```

```json
{
  "id": 40123,
  "status": "failed",
  "error": {
    "code": "PASSWORD_PROTECTED_PDF",
    "message": "PDF file is password-protected. Please remove the password and try again."
  },
  "portfolio_id": null,
  "closed_at": "2025-03-16T10:15:22Z"
}
```

### Recovery Steps

1. **Open PDF in Preview (Mac) or Adobe Acrobat**
2. **Enter the password to unlock it**
3. **Save as a new file without password protection:**
   - Mac Preview: File → Export as PDF... (uncheck "Encrypt")
   - Adobe Acrobat: File → Save As... → Security Method: "No Security"
4. **Re-upload the unprotected file:**
   ```bash
   powder --json upload ~/Documents/statement-q4-2024-unlocked.pdf
   ```

---

## Scenario 2: Unsupported File Type

### What Happened

Tried to upload a CSV file, which is not supported.

### Error Output

```bash
powder --json upload ~/Downloads/holdings.csv
```

```json
{
  "error": {
    "code": "UNSUPPORTED_FILE_TYPE",
    "message": "File type not supported. Supported formats: .pdf, .xlsx, .png, .jpg, .jpeg"
  }
}
```

### Recovery Steps

1. **Check file type:**
   ```bash
   file ~/Downloads/holdings.csv
   # Output: holdings.csv: ASCII text
   ```

2. **Convert to supported format:**
   - **Option A: Excel**
     - Open CSV in Excel or LibreOffice Calc
     - Save As → Excel Workbook (.xlsx)
   
   - **Option B: PDF (if from web)**
     - Open source webpage
     - Print to PDF
   
   - **Option C: Screenshot (if simple)**
     - Take screenshot of the data
     - Save as .png or .jpg

3. **Re-upload converted file:**
   ```bash
   powder --json upload ~/Downloads/holdings.xlsx
   ```

**Note:** For best results, use Excel (.xlsx) for tabular data or PDF for formatted statements.

---

## Scenario 3: Authentication Failure

### What Happened

`POWDER_API_TOKEN` environment variable not set or expired.

### Error Output

```bash
powder --json upload ~/Documents/statement.pdf
```

```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Authentication required. Please set POWDER_API_TOKEN environment variable."
  }
}
```

### Recovery Steps

1. **Check if token is set:**
   ```bash
   echo $POWDER_API_TOKEN
   # Output: (blank or old token)
   ```

2. **Get a new token:**
   - Log in to Powder dashboard
   - Navigate to Settings → API Tokens
   - Generate new token or copy existing one

3. **Set the environment variable:**
   
   **For current session:**
   ```bash
   export POWDER_API_TOKEN=your_token_here
   ```
   
   **Permanently (add to ~/.zshrc or ~/.bashrc):**
   ```bash
   echo 'export POWDER_API_TOKEN=your_token_here' >> ~/.zshrc
   source ~/.zshrc
   ```

4. **Verify token is set:**
   ```bash
   echo $POWDER_API_TOKEN
   # Output: your_token_here
   ```

5. **Retry upload:**
   ```bash
   powder --json upload ~/Documents/statement.pdf
   ```

---

## Scenario 4: Processing Timeout

### What Happened

Large or complex document took longer than expected to process.

### Error Output

```bash
powder --json status 41500 --watch --timeout 300
```

After 5 minutes (300 seconds):

```
Error: Timeout reached (300s). Upload may still be processing.
Current status: processing
```

### Recovery Steps

1. **Check status manually (without --watch):**
   ```bash
   powder --json status 41500
   ```
   
   ```json
   {
     "id": 41500,
     "status": "processing",
     "portfolio_id": null
   }
   ```

2. **Wait longer with increased timeout:**
   ```bash
   powder --json status 41500 --watch --timeout 900
   ```
   
   This will wait up to 15 minutes (900 seconds).

3. **If still processing after 15+ minutes:**
   - Check for errors without `--watch`:
     ```bash
     powder --json status 41500
     ```
   
   - If `status: "processing"` persists beyond 30 minutes, contact support with upload ID

4. **Once completed:**
   ```bash
   powder --json status 41500
   ```
   
   ```json
   {
     "id": 41500,
     "status": "done",
     "portfolio_id": 45200,
     "closed_at": "2025-03-16T10:42:15Z"
   }
   ```

5. **Retrieve data:**
   ```bash
   powder --json data 41500
   ```

**Typical processing times:**
- Simple statements (1-5 holdings): 15-30 seconds
- Standard statements (10-50 holdings): 30-90 seconds
- Complex statements (100+ holdings, multiple pages): 2-5 minutes
- Very large files (40+ MB, 50+ pages): 5-15 minutes

**Recommendation:** Use `--timeout 600` (10 minutes) as default for `--watch` commands.

---

## General Recovery Checklist

When an upload fails:

1. **Read the error message carefully** - it usually tells you exactly what's wrong
2. **Check error code** in [`error-codes.md`](../reference/error-codes.md)
3. **Verify file prerequisites:**
   - Supported format (.pdf, .xlsx, .png, .jpg, .jpeg)
   - Under 50MB
   - Not password-protected
   - Not corrupted (opens locally)
   - Readable permissions
4. **Check authentication:** `POWDER_API_TOKEN` is set and valid
5. **Fix the issue** and re-upload (uploads are idempotent - safe to retry)
6. **Contact support** if issue persists with upload ID and error details
