# Setup Guide

Complete setup instructions for connecting the nuph.ai MCP Server to your AI client.

## Before you start

1. **Get your nuph.ai API key** at [app.nuph.ai/extensions](https://app.nuph.ai/extensions)
   - Sign in to your nuph.ai account (free signup at [nuph.ai](https://nuph.ai))
   - Click "Create API key"
   - Copy the key (starts with `outreach_`)

2. **Keep your API key secret** — treat it like a password. It grants access to all your nuph.ai data.

---

## Claude.ai (web chat at claude.ai)

Claude.ai does **not** support custom Authorization headers on MCP connectors. Use the API key as a URL query parameter:

1. Open [claude.ai](https://claude.ai)
2. Click your profile picture → **Settings**
3. Go to **Connectors** (or "Integrations")
4. Click **Add custom connector**
5. Fill in:
   - **Name:** `nuph.ai`
   - **URL:**
     ```
     https://api.nuph.ai/functions/v1/outreach-mcp?apiKey=YOUR_API_KEY
     ```
     (Replace `YOUR_API_KEY` with your actual nuph.ai key, starts with `outreach_`.)
6. Click **Save**

The connector will show 47 tools available immediately. Try: *"Give me an overview of my nuph.ai workspace"* (calls `get_workspace_overview` — best first call).

**Supported query parameter names** (any of these works): `apiKey`, `apikey`, `api_key`, `key`, `token`.

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

- *"What nuph.ai tools do you have?"* — Should list all 47 tools
- *"Give me an overview of my workspace"* — Should call `get_workspace_overview` and return companies, active campaigns, credits, recent jobs, alerts
- *"List my companies"* — Should return your companies
- *"What's my credit balance?"* — Should return balance and plan
- *"Show me ALL my connected prospects"* — Should call `query_prospects` and return the real total (not capped at 50)

If you see errors, check [TROUBLESHOOTING.md](TROUBLESHOOTING.md).

## What to expect on the first write

Every **write tool** (`create_*`, `update_*`, `delete_*`, `search_leads`, `enrich_prospects`, `generate_*`, `audit_*`, `scrape_*`, `enhance_*`, `cancel_job`) defaults to `dry_run: true`. The first call returns a structured **preview**, not a write:

```jsonc
{
  "preview": { /* the row that would be written */ },
  "missing_required": [],
  "missing_recommended": [
    { "field": "icp_profile_id", "help": "Link to an existing ICP profile",
      "options_ref": "list_icp_profiles" }
  ],
  "warnings": [],
  "estimated_credits": null,
  "ready_to_execute": true,
  "next_call": {
    "tool": "create_campaign",
    "arguments": { /* echo with defaults */ },
    "with": { "dry_run": false }
  }
}
```

Your AI client uses this to ask you smart questions ("Which ICP profile should I link?") before actually writing. To skip the preview and execute, the model passes `dry_run: false` in the next call. **No accidental creates, no surprise credit spends.**

## What to expect with multiple companies

Tools that take `company_id` (e.g. `list_campaigns`, `query_prospects`, `get_dashboard_metrics`) auto-resolve it when you own exactly **one** company. If you own **more than one**, the response is a structured `ambiguous_company` error containing the option list — your AI client uses it to ask "Which company?" before retrying.

If you own **zero** companies, the response is `no_company` with a hint to call `create_company` first.

## Security best practices

- **Don't commit API keys** to public repos (use environment variables)
- **Revoke unused keys** regularly from the dashboard
- **Use separate keys** for different clients if you want granular audit trails
- **Monitor usage** at [app.nuph.ai/extensions](https://app.nuph.ai/extensions) — every key shows `last_used_at`

## Next steps

- [Tools Reference](TOOLS.md) — Learn what each tool does
- [Example Prompts](EXAMPLES.md) — 50+ tested prompts
- [Protocol Reference](PROTOCOL.md) — Low-level JSON-RPC details
