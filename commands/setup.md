---
description: Set up Powder — installs the CLI and verifies your API token.
---

You are setting up the Powder CLI for the user.

This command walks through installation, API token configuration, and validation.

## Setup Workflow

Follow these steps in order:

### 1. Check if `powder` CLI is Installed

Run: `which powder`

- **If found**: Verify the version with `powder --version` and show it to the user
- **If not found**: Install it with `pip install powder-cli`
  - After installation, verify: `powder --version`
  - If `pip` is not available, suggest: "You need Python and pip installed. Visit https://www.python.org/downloads/"

### 2. Check for API Token

Run: `echo $POWDER_API_TOKEN | head -c 10`

- **If set** (output is non-empty): Proceed to validation (step 3)
- **If not set**: Provide instructions:

```
Your Powder API token is not configured.

To get a token:
1. Contact support@powderfi.com and request an API token
2. Once you receive it, run:
   export POWDER_API_TOKEN=your_token_here

3. Add it to your shell profile to persist across sessions:
   echo 'export POWDER_API_TOKEN=your_token_here' >> ~/.bashrc   # for bash
   echo 'export POWDER_API_TOKEN=your_token_here' >> ~/.zshrc    # for zsh

4. Reload your shell or run: source ~/.bashrc (or ~/.zshrc)
```

Stop here and wait for the user to set the token before proceeding.

### 3. Validate API Token

Run: `powder --json status 0`

This makes a test API call. Interpret the response:

- **400 Bad Request**: ✓ Authentication works (the upload ID doesn't exist, but auth succeeded)
- **401 Unauthorized**: ✗ Invalid or expired token — tell the user to contact support@powderfi.com
- **Other errors**: Report the error and suggest contacting support

### 4. Print Setup Summary

Once all checks pass, provide a summary:

```
✓ Powder CLI Setup Complete

✓ powder CLI installed (version X.X.X)
✓ API token configured
✓ API connection verified

You're ready to upload financial statements!

Try: /Powder:upload path/to/statement.pdf
```

## Troubleshooting

If any step fails, provide specific guidance:

- **`pip install` fails**: Suggest checking Python installation or using `pip3 install powder-cli`
- **Token validation fails**: Suggest contacting support@powderfi.com with the error message
- **Command not found after install**: Suggest adding pip's bin directory to PATH or using `python -m powder`

## Example Output

For a fresh setup:

```
Setting up Powder CLI...

✓ powder CLI installed (version 1.2.3)
⚠ API token not set

To get a token:
1. Contact support@powderfi.com and request an API token
2. Once you receive it, run:
   export POWDER_API_TOKEN=your_token_here

3. Add it to your shell profile:
   echo 'export POWDER_API_TOKEN=your_token_here' >> ~/.zshrc

After setting your token, run /Powder:setup again to validate.
```

For an existing setup:

```
✓ Powder CLI Setup Complete

✓ powder CLI installed (version 1.2.3)
✓ API token configured
✓ API connection verified

You're ready to upload financial statements!

Try: /Powder:upload path/to/statement.pdf
```
