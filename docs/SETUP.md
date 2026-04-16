# Setup Guide

Complete setup instructions for connecting the nuph.ai MCP Server to your AI client.

## Before you start

1. **Get your nuph.ai API key** at [app.nuph.ai/extensions](https://app.nuph.ai/extensions)
   - Sign in to your nuph.ai account (free signup at [nuph.ai](https://nuph.ai))
   - Click "Create API key"
   - Copy the key (starts with `outreach_`)

2. **Keep your API key secret** — treat it like a password. It grants access to all your nuph.ai data.

---

## Claude Desktop (macOS)

1. Open Terminal
2. Edit the config file:
   ```bash
   open ~/Library/Application\ Support/Claude/claude_desktop_config.json
   ```
   If the file doesn't exist, create it first:
   ```bash
   mkdir -p ~/Library/Application\ Support/Claude
   touch ~/Library/Application\ Support/Claude/claude_desktop_config.json
   ```

3. Add this configuration (replace `YOUR_API_KEY`):
   ```json
   {
     "mcpServers": {
       "nuph": {
         "transport": {
           "type": "streamable-http",
           "url": "https://api.nuph.ai/functions/v1/outreach-mcp"
         },
         "headers": {
           "Authorization": "Bearer YOUR_API_KEY"
         }
       }
     }
   }
   ```

4. Quit Claude Desktop completely (Cmd+Q) and relaunch
5. Start a new conversation and ask: *"What nuph.ai tools do you have available?"*

## Claude Desktop (Windows)

1. Open File Explorer and navigate to: `%APPDATA%\Claude\`
2. Open `claude_desktop_config.json` (create it if it doesn't exist)
3. Add the same configuration as above
4. Restart Claude Desktop

## Claude Desktop (Linux)

1. Edit `~/.config/Claude/claude_desktop_config.json`
2. Add the same configuration
3. Restart Claude Desktop

## Claude Code (CLI)

One-line setup:

```bash
claude mcp add nuph --transport http "https://api.nuph.ai/functions/v1/outreach-mcp" --header "Authorization: Bearer YOUR_API_KEY"
```

Verify it's configured:

```bash
claude mcp list
```

To remove:

```bash
claude mcp remove nuph
```

## Cursor

1. Open Cursor Settings (`Cmd+,` on macOS, `Ctrl+,` on Windows/Linux)
2. Search for "MCP"
3. Click "Edit in settings.json"
4. Add:
   ```json
   {
     "mcp": {
       "servers": {
         "nuph": {
           "url": "https://api.nuph.ai/functions/v1/outreach-mcp",
           "headers": {
             "Authorization": "Bearer YOUR_API_KEY"
           }
         }
       }
     }
   }
   ```
5. Restart Cursor

Alternatively, edit `~/.cursor/mcp.json` directly.

## Continue.dev

Add to your Continue config (`~/.continue/config.json`):

```json
{
  "mcpServers": {
    "nuph": {
      "transport": {
        "type": "streamable-http",
        "url": "https://api.nuph.ai/functions/v1/outreach-mcp"
      },
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY"
      }
    }
  }
}
```

## Zed Editor

Add to `~/.config/zed/settings.json`:

```json
{
  "context_servers": {
    "nuph": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://api.nuph.ai/functions/v1/outreach-mcp",
        "--header",
        "Authorization:Bearer YOUR_API_KEY"
      ]
    }
  }
}
```

(Zed currently requires stdio; we use `mcp-remote` as a proxy.)

## Any other MCP client

Just use:
- **URL:** `https://api.nuph.ai/functions/v1/outreach-mcp`
- **Transport:** Streamable HTTP
- **Header:** `Authorization: Bearer YOUR_API_KEY`

If your client only supports stdio, use the `mcp-remote` proxy:

```bash
npx -y mcp-remote https://api.nuph.ai/functions/v1/outreach-mcp --header "Authorization:Bearer YOUR_API_KEY"
```

## Verify the connection

After setup, ask your AI:

- *"What nuph.ai tools do you have?"* — Should list all 9 tools
- *"List my companies"* — Should return your companies
- *"What's my credit balance?"* — Should return balance and plan

If you see errors, check [TROUBLESHOOTING.md](TROUBLESHOOTING.md).

## Security best practices

- **Don't commit API keys** to public repos (use environment variables)
- **Revoke unused keys** regularly from the dashboard
- **Use separate keys** for different clients if you want granular audit trails
- **Monitor usage** at [app.nuph.ai/extensions](https://app.nuph.ai/extensions) — every key shows `last_used_at`

## Next steps

- [Tools Reference](TOOLS.md) — Learn what each tool does
- [Example Prompts](EXAMPLES.md) — 50+ tested prompts
- [Protocol Reference](PROTOCOL.md) — Low-level JSON-RPC details
