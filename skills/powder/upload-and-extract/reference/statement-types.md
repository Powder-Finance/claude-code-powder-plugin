# Statement Types

The `--type` flag specifies the type of financial document being uploaded.

## Available Types

Currently, only one statement type is supported:

### `brokerage` (Default)

Brokerage account statements, including:
- **401(k) statements** (Fidelity, Vanguard, Charles Schwab, etc.)
- **IRA statements** (Traditional IRA, Roth IRA, SEP IRA)
- **Individual brokerage accounts**
- **Investment account statements**
- **Portfolio holdings reports**

**This is the default type** - if you don't specify `--type`, `brokerage` is assumed.

---

## Usage

### Explicit Type Specification

```bash
powder --json upload ~/Documents/statement.pdf --type brokerage
```

### Default Behavior (Recommended)

Since `brokerage` is the only supported type and is the default, you can omit the `--type` flag:

```bash
powder --json upload ~/Documents/statement.pdf
```

Both commands above are equivalent.

---

## Examples

**401(k) statement:**
```bash
powder --json upload ~/Documents/fidelity-401k-2024.pdf
```

**IRA statement:**
```bash
powder --json upload ~/Documents/vanguard-ira.pdf --type brokerage
```

**Individual brokerage:**
```bash
powder --json upload ~/Documents/schwab-account.pdf
```

**Excel export of holdings:**
```bash
powder --json upload ~/Downloads/portfolio-export.xlsx
```

---

## Future Statement Types

Additional statement types may be added in future releases, such as:
- Bank statements
- Credit card statements  
- Loan documents
- Tax documents

When new types are available, they will be documented here with specific usage examples.

---

## What Gets Extracted

For `brokerage` type statements, the Powder API extracts:

**Holdings (Ownerships):**
- Security name, ticker, ISIN, CUSIP
- Quantity, price, market value
- Account details (name, type)
- Asset classification
- Performance metrics (gain/loss, yield, expense ratio)

**Account Information:**
- Account name and type (401k, IRA, Individual, etc.)
- Account allocation percentages
- Portfolio totals

See the data retrieval skill for complete field reference.

---

## Portfolio Association

You can optionally associate an upload with a specific portfolio using `--portfolio`:

```bash
powder --json upload ~/Documents/statement.pdf --portfolio 12345
```

This links the extracted data to portfolio ID `12345`. If omitted, a new portfolio is created automatically.

---

## Notes

- **Auto-detection is not available** - the `--type` flag does not support `auto` or auto-detection
- All uploads currently use the `brokerage` type
- The API intelligently parses different brokerage statement formats (Fidelity, Vanguard, Schwab, etc.)
- No need to specify the brokerage provider - the system detects it automatically

---

## Error Handling

If you attempt to use an unsupported type:

```bash
powder --json upload ~/Documents/bank.pdf --type bank
```

You'll receive an error:
```json
{
  "error": {
    "code": "INVALID_TYPE",
    "message": "Invalid statement type. Supported types: brokerage"
  }
}
```

**Solution:** Use `--type brokerage` or omit the flag entirely.
