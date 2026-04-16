# Troubleshooting

Common issues and fixes for the nuph.ai MCP Server.

## Claude doesn't see the nuph tools

**Symptoms:** You ask "What nuph.ai tools do you have?" and Claude says it can't find any.

**Checks:**

1. **Did you restart Claude Desktop?** — Config changes require a full quit (Cmd+Q on Mac) and relaunch.

2. **Is the JSON valid?** — Paste your `claude_desktop_config.json` into [jsonlint.com](https://jsonlint.com) to verify.

3. **Is the transport type correct?** — It should be `"streamable-http"`, not `"http"` or `"stream"`.
   ```json
   "transport": { "type": "streamable-http", "url": "..." }
   ```

4. **Check Claude Desktop logs** (macOS): `~/Library/Logs/Claude/mcp-server-nuph.log`

## "401 Unauthorized" or "API key required"

**Symptoms:** Tool calls fail with auth errors.

**Checks:**

1. **Is the key correct?** — Copy it fresh from [app.nuph.ai/extensions](https://app.nuph.ai/extensions).
2. **Does it start with `outreach_`?** — All valid keys have this prefix.
3. **Is the header format right?** — Must be `Authorization: Bearer YOUR_KEY` (with "Bearer " prefix).
4. **Is the key active?** — Check the dashboard. Keys can be revoked.
5. **Is the key expired?** — Keys can optionally have expiration dates.

## "Invalid or inactive API key"

1. Go to [app.nuph.ai/extensions](https://app.nuph.ai/extensions)
2. If the key is inactive, reactivate or create a new one
3. Update your MCP client config
4. Restart the client

## Tool call returns "access denied" for my own data

**Symptoms:** You're trying to get a prospect/campaign you own but get "not found or access denied".

**Checks:**

1. **Are you using the right API key?** — If you have multiple nuph.ai accounts, each has different keys.
2. **Does the resource belong to you?** — The MCP verifies ownership on every call.
3. **Is the ID correct?** — Prospect, campaign, and company IDs are UUIDs. Copy-paste carefully.

## Searches don't return results

**Symptoms:** `search_leads` succeeds but no new prospects appear.

**Checks:**

1. **Did you wait 1-3 minutes?** — Searches are asynchronous.
2. **Check the campaign in the dashboard** — Results appear in the prospect list.
3. **Might be 100% duplicates** — If leads are already in your pipeline, MongoDB dedup blocks re-scraping. Credits are refunded.
4. **Check your ICP filters** — Very narrow filters (e.g. very specific titles + locations) may return few results.

## "Credits deducted but no leads"

All searches cost a minimum of 1 credit (the search page fee) even if 0 new leads are found. This is a real Apify cost we pass through. If all leads were duplicates of your existing pipeline, you only pay the 1-credit search fee.

## Cursor can't connect

**Symptoms:** Cursor UI shows MCP server is disconnected.

**Checks:**

1. **Cursor MCP support requires recent version** — Update Cursor.
2. **Check settings.json has the right structure**:
   ```json
   {
     "mcp": {
       "servers": {
         "nuph": { "url": "...", "headers": {...} }
       }
     }
   }
   ```
3. **Restart Cursor** after config changes.

## Slow responses

**Symptoms:** Tool calls take a long time.

**Expected:**
- Read-only queries (`list_*`, `get_*`): <1 second
- `search_leads`: returns job_id immediately, but actual search takes 1-3 minutes
- `generate_message`: 2-5 seconds (AI generation)

**If slower:**
- Check [nuph.ai/status/](https://nuph.ai/status/) for outages
- Check your own network connection
- Rate limiting may be slowing you down if you're hitting the server heavily

## "Method not found"

**Symptoms:** JSON-RPC error code `-32601`.

**Cause:** Calling an unsupported method or wrong tool name.

**Supported methods:**
- `initialize`
- `notifications/initialized`
- `tools/list`
- `tools/call`
- `ping`

**Supported tool names:** (9 total)
`list_companies`, `get_credits_balance`, `list_campaigns`, `get_campaign`, `list_prospects`, `get_prospect`, `search_leads`, `generate_message`, `list_recent_activity`

Note: There is NO `approve_prospect` tool (intentional — approvals must be done from the dashboard).

## Corporate firewall blocks the endpoint

**Symptoms:** Can't reach `api.nuph.ai`.

**Fix:** Whitelist `*.nuph.ai` in your corporate firewall. The MCP endpoint is `https://api.nuph.ai/functions/v1/outreach-mcp`.

## "I want a feature that's not supported"

Read the [Tools Reference](TOOLS.md) for what's intentionally not supported (approve, send, delete, billing).

For legitimate feature requests, [open a discussion](https://github.com/nuph-ai/nuph-mcp-server/discussions).

## Still stuck?

Email [here@nuph.ai](mailto:here@nuph.ai) with:
- Your MCP client and version
- The exact error message
- Steps to reproduce
- Your nuph.ai account email (so we can look up logs)

We usually respond within 24 hours.
