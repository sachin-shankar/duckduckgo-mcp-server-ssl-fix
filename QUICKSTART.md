# Quick Start Guide

Get the DuckDuckGo MCP server working in Claude Code in 5 minutes!

## TL;DR

```bash
# 1. Test locally first (from this directory)
DDG_DISABLE_SSL_VERIFY=true uv run python test_search.py --search "test" --max-results 2

# 2. Option A: Use locally without publishing
uv pip install -e .

# 2. Option B: Publish to PyPI (requires PyPI account)
uv build
uv run twine upload dist/*

# 3. Configure Claude Code
mkdir -p ~/.config/claude-code
cat > ~/.config/claude-code/settings.json << 'EOF'
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
EOF

# 4. Restart Claude Code and test!
```

## Detailed Steps

### Step 1: Verify the Package Works

```bash
cd /Users/smandyashankar/Downloads/duckduckgo-mcp-server

# Test with SSL verification disabled
DDG_DISABLE_SSL_VERIFY=true uv run python test_search.py --all
```

Expected output: ✓ Success messages with search results

### Step 2: Choose Your Installation Method

#### Option A: Local Installation (No Publishing Required)

Best for: Personal use, testing, or if you don't want to publish publicly

```bash
# Install in editable mode
uv pip install -e .

# Configure Claude Code to use local installation
cat > ~/.config/claude-code/settings.json << 'EOF'
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
EOF
```

#### Option B: Publish to PyPI

Best for: Sharing with others, easier updates, cleaner setup

See [PUBLISHING_GUIDE.md](PUBLISHING_GUIDE.md) for detailed instructions.

Quick version:
```bash
# Build
uv build

# Publish (requires PyPI account and API token)
uv run twine upload dist/*

# Configure Claude Code to use published package
cat > ~/.config/claude-code/settings.json << 'EOF'
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
EOF
```

### Step 3: Test in Claude Code

1. Restart Claude Code
2. Open a new conversation
3. Ask: "Search DuckDuckGo for Python tutorials"
4. Claude should use the `mcp__duckduckgo__search` tool automatically

### Step 4: Verify It's Working

You should see Claude Code using these tools:
- `mcp__duckduckgo__search` - For web searches
- `mcp__duckduckgo__fetch_content` - For fetching webpage content

## Configuration Options

### SSL Verification Modes

| Mode | Configuration | Use Case |
|------|--------------|----------|
| **Disabled** | `"DDG_DISABLE_SSL_VERIFY": "true"` | Corporate proxy with SSL inspection |
| **Custom Cert** | `"DDG_SSL_CERT_PATH": "/path/to/cert.pem"` | You have your org's certificate |
| **Default** | No env vars | Normal internet connection |

### Example Configurations

**For Corporate Proxies (most common):**
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

**With Custom Certificate:**
```json
{
  "mcpServers": {
    "duckduckgo": {
      "command": "uvx",
      "args": ["duckduckgo-mcp-server-ssl-fix"],
      "env": {
        "DDG_SSL_CERT_PATH": "/path/to/corporate-cert.pem"
      }
    }
  }
}
```

**Default (No SSL Issues):**
```json
{
  "mcpServers": {
    "duckduckgo": {
      "command": "uvx",
      "args": ["duckduckgo-mcp-server-ssl-fix"]
    }
  }
}
```

## Troubleshooting

### "No results found" or Empty Results

**Problem**: SSL verification failing silently

**Solution**: Make sure `DDG_DISABLE_SSL_VERIFY` is set in configuration

### "Command not found: duckduckgo-mcp-server-ssl-fix"

**Problem**: Package not installed or not published

**Solutions**:
1. Local install: `uv pip install -e .`
2. Or publish to PyPI first
3. Or use `uvx` which auto-installs: `"command": "uvx"`

### Claude Code Not Using the Tool

**Check**:
1. Configuration file location: `~/.config/claude-code/settings.json`
2. Restart Claude Code after config changes
3. Verify package works: `DDG_DISABLE_SSL_VERIFY=true uvx duckduckgo-mcp-server-ssl-fix`

### Testing Without Claude Code

Use the test script to verify everything works:

```bash
# Test search
DDG_DISABLE_SSL_VERIFY=true uv run python test_search.py --search "your query"

# Test content fetching
DDG_DISABLE_SSL_VERIFY=true uv run python test_search.py --fetch "https://example.com"

# Run all tests
DDG_DISABLE_SSL_VERIFY=true uv run python test_search.py --all
```

## Files Reference

- `README.md` - Full documentation
- `PUBLISHING_GUIDE.md` - Detailed publishing instructions
- `TESTING_GUIDE.md` - Original SSL issue analysis
- `claude-code-config-example.json` - Example configuration
- `test_search.py` - Test script

## What Changed from Original?

This fork adds:
1. **Configurable SSL verification** via `DDG_DISABLE_SSL_VERIFY` environment variable
2. **Custom certificate support** via `DDG_SSL_CERT_PATH` environment variable
3. **Better error visibility** - actually shows SSL errors instead of "no results"
4. **Corporate proxy friendly** - works behind SSL-inspecting firewalls

## Next Steps

Once working:
1. ✅ You can search DuckDuckGo from Claude Code
2. ✅ You can fetch and parse web content
3. ✅ Works despite SSL inspection/corporate proxies
4. ✅ Share with colleagues who have the same issues

## Need Help?

1. Check the test script output for actual errors
2. Review `TESTING_GUIDE.md` for SSL issue details
3. Review `PUBLISHING_GUIDE.md` for publishing help
4. Check Claude Code logs for MCP server errors

Enjoy! 🎉
