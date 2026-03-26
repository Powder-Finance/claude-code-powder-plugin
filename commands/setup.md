---
description: Set up Powder — installs the CLI and verifies your API token.
---

You are setting up the Powder CLI for the user.

This command walks through installation, API token configuration, and validation.

## Important: Token Resolution

The API token can come from two sources. Always check both:
1. **Plugin config** (set during plugin install): available as `CLAUDE_PLUGIN_OPTION_POWDER_API_TOKEN`
2. **Environment variable**: `POWDER_API_TOKEN`

When running any `powder` CLI command, ensure the token is passed through:

```bash
POWDER_API_TOKEN="${POWDER_API_TOKEN:-$CLAUDE_PLUGIN_OPTION_POWDER_API_TOKEN}" powder --json status 0
```

This way the CLI works whether the user configured their token via the plugin install prompt or via an environment variable.

## Setup Workflow

Follow these steps in order:

### 1. Check if `powder` CLI is Installed

Run: `which powder`

- **If found**: Verify the version with `powder --version` and show it to the user
- **If not found**: Install it with `pip install powder-cli`
  - After installation, verify: `powder --version`
  - If `pip` is not available, suggest: "You need Python and pip installed. Visit https://www.python.org/downloads/"

### 2. Check for API Token

Run: `test -n "${POWDER_API_TOKEN:-$CLAUDE_PLUGIN_OPTION_POWDER_API_TOKEN}" && echo "set" || echo "not set"`

- **If set**: Proceed to validation (step 3)
- **If not set**: Tell the user:

```
Your Powder API token is not configured.

To set it up, go to your plugin settings and add your API key,
or contact support@powderfi.com to get one.
```

Stop here and wait for the user to configure their token.

### 3. Validate API Token

Run: `POWDER_API_TOKEN="${POWDER_API_TOKEN:-$CLAUDE_PLUGIN_OPTION_POWDER_API_TOKEN}" powder --json status 0`

This makes a test API call. Interpret the response:

- **400 Bad Request**: ✓ Authentication works (the upload ID doesn't exist, but auth succeeded)
- **401 Unauthorized**: ✗ Invalid or expired token — tell the user to contact support@powderfi.com
- **Other errors**: Report the error and suggest contacting support

### 4. Print Setup Summary

Once all checks pass, provide a summary:

```
✓ Powder Setup Complete

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
