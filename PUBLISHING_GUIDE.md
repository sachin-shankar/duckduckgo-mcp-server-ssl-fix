# Publishing Guide for duckduckgo-mcp-server-ssl-fix

This guide will walk you through publishing your package to PyPI and using it in Claude Code.

## Prerequisites

1. **PyPI Account**: Create an account at https://pypi.org
2. **PyPI API Token**: Generate an API token from your PyPI account settings
3. **uv or pip**: Package manager installed

## Step 1: Prepare for Publishing

### 1.1 Update Your Email (Optional)

Edit `pyproject.toml` and update the author email:

```toml
authors = [{ name = "Smandya Shankar", email = "your-real-email@example.com" }]
```

### 1.2 Verify Package Build

```bash
cd /Users/smandyashankar/Downloads/duckduckgo-mcp-server
uv build
```

This should create two files in `dist/`:
- `duckduckgo_mcp_server_ssl_fix-0.2.0.tar.gz` (source distribution)
- `duckduckgo_mcp_server_ssl_fix-0.2.0-py3-none-any.whl` (wheel)

## Step 2: Publish to PyPI

### Option 1: Using `uv` (Recommended)

```bash
# Install twine if not already installed
uv pip install twine

# Upload to PyPI
uv run twine upload dist/*
```

When prompted:
- Username: `__token__`
- Password: Your PyPI API token (starts with `pypi-`)

### Option 2: Using `twine` directly

```bash
# Install twine
pip install twine

# Upload to PyPI
python -m twine upload dist/*
```

### Option 3: Test on TestPyPI First (Recommended for first-time publishers)

```bash
# Upload to TestPyPI
uv run twine upload --repository testpypi dist/*

# Test installation from TestPyPI
uv pip install --index-url https://test.pypi.org/simple/ duckduckgo-mcp-server-ssl-fix
```

## Step 3: Verify Publication

Once published, verify at:
- https://pypi.org/project/duckduckgo-mcp-server-ssl-fix/

## Step 4: Install and Test from PyPI

```bash
# Install from PyPI
uv pip install duckduckgo-mcp-server-ssl-fix

# Or use with uvx (no installation needed)
uvx duckduckgo-mcp-server-ssl-fix
```

## Step 5: Configure in Claude Code

### Method 1: Global Configuration (Recommended)

Edit or create `~/.config/claude-code/settings.json`:

```json
{
  "mcpServers": {
    "duckduckgo": {
      "command": "uvx",
      "args": ["duckduckgo-mcp-server-ssl-fix"],
      "env": {
        "DDG_DISABLE_SSL_VERIFY": "true"
      }
    }
  }
}
```

### Method 2: Project-Specific Configuration

Create `.claude/settings.json` in your project:

```json
{
  "mcpServers": {
    "duckduckgo": {
      "command": "uvx",
      "args": ["duckduckgo-mcp-server-ssl-fix"],
      "env": {
        "DDG_DISABLE_SSL_VERIFY": "true"
      }
    }
  }
}
```

### Method 3: Using Custom Certificate

If you have your organization's certificate:

```json
{
  "mcpServers": {
    "duckduckgo": {
      "command": "uvx",
      "args": ["duckduckgo-mcp-server-ssl-fix"],
      "env": {
        "DDG_SSL_CERT_PATH": "/path/to/your/cert.pem"
      }
    }
  }
}
```

## Step 6: Test in Claude Code

1. Restart Claude Code (or reload the MCP servers)
2. In Claude Code, try using the search tool:
   - Type a message like "Search for Python tutorials"
   - Claude should automatically use the `mcp__duckduckgo__search` tool

## Troubleshooting

### Build Errors

If `uv build` fails:
```bash
# Clean dist directory and rebuild
rm -rf dist/
uv build
```

### Upload Errors

**Error: "File already exists"**
- Increment version in `pyproject.toml` (e.g., `0.2.0` → `0.2.1`)
- Rebuild: `uv build`
- Upload again

**Error: "Invalid credentials"**
- Verify you're using `__token__` as username
- Check your API token is correct and has upload permissions

### Claude Code Not Finding the Tool

1. Check MCP server configuration:
   ```bash
   cat ~/.config/claude-code/settings.json
   ```

2. Check Claude Code logs for errors

3. Verify the package is accessible:
   ```bash
   uvx duckduckgo-mcp-server-ssl-fix --help
   ```

### SSL Still Not Working

1. Verify environment variable is set:
   ```json
   "env": {
     "DDG_DISABLE_SSL_VERIFY": "true"
   }
   ```

2. Try running directly to test:
   ```bash
   DDG_DISABLE_SSL_VERIFY=true uvx duckduckgo-mcp-server-ssl-fix
   ```

## Alternative: Use Without Publishing (Local Installation)

If you don't want to publish to PyPI, you can use the local package:

### Install Locally
```bash
cd /Users/smandyashankar/Downloads/duckduckgo-mcp-server
uv pip install -e .
```

### Configure Claude Code to Use Local Installation

```json
{
  "mcpServers": {
    "duckduckgo": {
      "command": "python",
      "args": ["-m", "duckduckgo_mcp_server.server"],
      "env": {
        "DDG_DISABLE_SSL_VERIFY": "true"
      }
    }
  }
}
```

Or use the script name directly:

```json
{
  "mcpServers": {
    "duckduckgo": {
      "command": "duckduckgo-mcp-server-ssl-fix",
      "env": {
        "DDG_DISABLE_SSL_VERIFY": "true"
      }
    }
  }
}
```

## Updating the Package

When you make changes:

1. Update version in `pyproject.toml`:
   ```toml
   version = "0.2.1"  # or 0.3.0 for features, 1.0.0 for major changes
   ```

2. Rebuild:
   ```bash
   rm -rf dist/
   uv build
   ```

3. Upload:
   ```bash
   uv run twine upload dist/*
   ```

4. Users can upgrade:
   ```bash
   uv pip install --upgrade duckduckgo-mcp-server-ssl-fix
   ```

## Version Numbering

Follow [Semantic Versioning](https://semver.org/):
- **0.2.1** (patch): Bug fixes, no new features
- **0.3.0** (minor): New features, backward compatible
- **1.0.0** (major): Breaking changes

## Getting Help

If you encounter issues:

1. Check the test script:
   ```bash
   DDG_DISABLE_SSL_VERIFY=true uv run python test_search.py --all
   ```

2. Review the README.md for configuration options

3. Check PyPI package page for any issues

## Success Checklist

- [ ] Package builds successfully (`uv build`)
- [ ] Package installs locally (`uv pip install ./dist/*.whl`)
- [ ] Test script works with built package
- [ ] Published to PyPI (or TestPyPI)
- [ ] Claude Code configuration created
- [ ] Tools work in Claude Code

## Next Steps

Once published and configured:
1. You can use the `search` tool in Claude Code
2. You can use the `fetch_content` tool in Claude Code
3. The tools will work despite SSL inspection/corporate proxies
4. You can share the package name with colleagues who have the same SSL issues

Enjoy your SSL-friendly DuckDuckGo MCP server! 🚀
