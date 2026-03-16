# Powder Claude Code Plugin

Upload financial documents to Powder for automated data extraction. This plugin integrates Powder's financial document processing API directly into Claude Code, enabling you to extract holdings, transactions, and account data from brokerage statements, account statements, and other financial documents.

## Features

- **4 Slash Commands** for quick access to Powder functionality
- **2 Skills** for automated document processing workflows
- **Automatic CLI Installation** via `/Powder:setup`
- **Support for Multiple File Types**: PDF, XLSX, PNG, JPG, JPEG (max 50MB)

## Commands

| Command | Description |
|---------|-------------|
| `/Powder:upload` | Upload a financial document for processing |
| `/Powder:data` | Retrieve extracted data from a processed document |
| `/Powder:status` | Check the processing status of an uploaded document |
| `/Powder:setup` | Install and configure the Powder CLI |

## Skills

- **upload-and-extract**: Automated workflow for uploading documents and polling until extraction completes
- **data-retrieval**: Fetch and format extracted holdings and transactions data

## Installation

### 1. Add the Powder Marketplace

```bash
/plugin marketplace add powderfi/claude-code-powder-plugin
```

### 2. Install the Plugin

```bash
/plugin install powder-plugin@powder-plugin-marketplace
```

### 3. Run Setup

```bash
/Powder:setup
```

This command will automatically install the Powder CLI if it's not already available.

### 4. Set Your API Token

Get your API token from support@powderfi.com, then set it in your environment:

```bash
export POWDER_API_TOKEN=your_token_here
```

Add this to your `~/.zshrc` or `~/.bashrc` to make it permanent.

### 5. Try It Out

```bash
/Powder:upload ~/documents/statement.pdf
```

Claude will upload your document, wait for processing to complete, and display the extracted data.

## Supported File Types

- **PDF** - Brokerage statements, account statements, tax documents
- **XLSX** - Excel-based financial reports
- **PNG, JPG, JPEG** - Scanned or photographed documents

**Maximum file size**: 50MB

## How It Works

1. **Upload**: Send your financial document to Powder's secure API
2. **Processing**: Powder's AI extracts holdings, transactions, and account information
3. **Retrieval**: Get structured JSON data with all extracted information
4. **Integration**: Use the data directly in Claude Code workflows

## Example Workflow

```bash
# Upload a statement
/Powder:upload ~/Downloads/fidelity-statement-q4-2024.pdf

# Check status
/Powder:status abc123

# Get the data
/Powder:data abc123
```

Or use the automated skill:

```bash
# Claude will handle upload + polling + retrieval automatically
"Upload and extract data from my Q4 statement at ~/Downloads/fidelity-statement-q4-2024.pdf"
```

## Troubleshooting

If you encounter issues:

1. Verify your API token is set: `echo $POWDER_API_TOKEN`
2. Check CLI installation: `which powder`
3. Run `/Powder:setup` to reinstall the CLI
4. Contact support@powderfi.com for API access issues

## Credits

Built by **Powder Financial** (https://powderfi.com)

For questions, support, or feedback: support@powderfi.com

