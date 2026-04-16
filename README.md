<div align="center">

# nuph.ai MCP Server

### Control your LinkedIn outreach from Claude, Cursor, or any MCP-compatible AI

[![MCP](https://img.shields.io/badge/Model%20Context%20Protocol-v2024--11--05-F5A623?style=for-the-badge)](https://modelcontextprotocol.io/)
[![Streamable HTTP](https://img.shields.io/badge/Transport-Streamable%20HTTP-CF7153?style=for-the-badge)](https://modelcontextprotocol.io/specification/2024-11-05/basic/transports)
[![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Production-10B981?style=for-the-badge)](https://nuph.ai/integrations/mcp/)

**[Website](https://nuph.ai)** · **[Documentation](https://nuph.ai/integrations/mcp/)** · **[llms.txt](https://nuph.ai/llms.txt)** · **[Get API Key](https://app.nuph.ai/extensions)**

---

**Connect [nuph.ai](https://nuph.ai) — the AI-powered LinkedIn outreach platform — to Claude Desktop, Claude Code, Cursor, Continue, or any MCP-compatible AI client. Search LinkedIn for leads, manage campaigns, and generate personalized AI messages through natural language.**

</div>

## What is this?

This is the public documentation and reference for the **nuph.ai MCP Server** — a remote [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) server that lets AI agents like [Claude](https://claude.ai) control the [nuph.ai LinkedIn outreach platform](https://nuph.ai) through natural language.

Think of it as "USB-C for AI agents": connect once, and any AI can now search your leads, check your campaigns, preview personalized messages, and query your credit balance — all through JSON-RPC over HTTP.

### Why nuph.ai + MCP?

- **Search LinkedIn from Claude** — "Find 25 CTOs in Spain, company size 51-200"
- **Query your pipeline** — "Show me my top 10 prospects with score above 8"
- **Preview AI messages** — "Generate a connection message for [prospect name]"
- **Check stats** — "How many credits do I have? What's my best performing campaign?"
- **Zero setup** — No local install, just paste a URL + API key into your MCP client

## Quick Start

### 1. Get your API key

Sign in at [app.nuph.ai/extensions](https://app.nuph.ai/extensions) and create a new API key. Copy it (it starts with `outreach_`).

Don't have a nuph.ai account? [Sign up free](https://nuph.ai) — 50 credits/month, no credit card.

### 2. Connect your AI client

#### Claude Desktop

Edit `claude_desktop_config.json`:

**macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
**Windows:** `%APPDATA%\Claude\claude_desktop_config.json`
**Linux:** `~/.config/Claude/claude_desktop_config.json`

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

Restart Claude Desktop.

#### Claude Code (CLI)

```bash
claude mcp add nuph --transport http "https://api.nuph.ai/functions/v1/outreach-mcp" --header "Authorization: Bearer YOUR_API_KEY"
```

#### Cursor

Add to `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "nuph": {
      "url": "https://api.nuph.ai/functions/v1/outreach-mcp",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY"
      }
    }
  }
}
```

#### Any other MCP client

- **Endpoint:** `https://api.nuph.ai/functions/v1/outreach-mcp`
- **Transport:** Streamable HTTP
- **Protocol:** JSON-RPC 2.0
- **Auth:** `Authorization: Bearer <your_api_key>`
- **Discovery:** `https://nuph.ai/.well-known/mcp.json`

### 3. Start asking

```
You: Find 25 VPs of Engineering in Berlin with companies sized 51-200

Claude:
  → search_leads(keywords: ["VP of Engineering"], locations: ["Berlin"], ...)
  ✓ Search launched. Results will appear in your prospects list in 1-3 minutes.

You: Show me my top 10 scoring prospects from this campaign

Claude:
  → list_prospects(min_score: 8, limit: 10)
  ✓ Returns 10 prospects with score 8+, sorted by match quality
```

## 9 Tools Available

| Tool | Description |
|------|-------------|
| `search_leads` | Search LinkedIn for new leads with advanced filters (seniority, function, company size, languages) |
| `list_prospects` | List prospects with filtering by campaign, status, or score |
| `get_prospect` | Get full prospect details including experience, skills, languages, and all generated messages |
| `list_campaigns` | List campaigns with status and stats |
| `get_campaign` | Get campaign details, ICP profile, and prospect stats |
| `generate_message` | Preview or regenerate an AI message for a prospect (does NOT send) |
| `list_companies` | List all companies you own |
| `get_credits_balance` | Check credit balance and plan info |
| `list_recent_activity` | Get recent activity log (approvals, messages, searches) |

**Important — what this MCP server does NOT do:**
- ❌ Cannot approve prospects (user must approve from dashboard after review)
- ❌ Cannot send LinkedIn messages or connection requests (requires the official [Chrome Extension](https://chromewebstore.google.com/detail/nuphai-linkedin-outreach/ekcjniemnbdjjobajommjdnoimhdijel))
- ❌ Cannot modify billing, plan, or account settings
- ❌ Cannot delete prospects, campaigns, or companies

The MCP is designed for **search, exploration, and message preview**. Actual outreach happens via the dashboard + Chrome extension for user safety.

## Example Prompts

Try these in your connected AI client:

- *"Show me my top 10 prospects with score above 8"*
- *"Search for 25 VP Engineering in Germany with company size 51-200"*
- *"List my active campaigns and their stats"*
- *"How many credits do I have left?"*
- *"Generate a preview connection message for prospect [ID]"*
- *"Show me recent activity from the last 20 actions"*
- *"Which of my campaigns has the most connected prospects?"*
- *"Find 50 CTOs in Spain who speak Spanish"*
- *"Search for Head of Sales in the US, company size 201-500, then show me the top scoring ones"*

## What is nuph.ai?

[nuph.ai](https://nuph.ai) is an AI-powered LinkedIn outreach platform for B2B sales teams, recruiters, and agencies. It automates the full outreach workflow:

1. **Discover** — Search LinkedIn with advanced filters
2. **Enrich** — Get verified emails + 50+ data points per profile
3. **Score** — AI scores every prospect 0-10 against your ICP
4. **Generate** — AI writes personalized connection notes and follow-up sequences
5. **Send** — Chrome extension sends connections directly from LinkedIn

**Plans:** Free ($0, 50 credits), Starter ($59, 1,000 credits), Pro ($129, 2,500 credits), Agency ($299, 5,000 credits).

## Security

- **API key authentication** on every request (Bearer token)
- **Ownership verification** — you can only access data in companies you own
- **Scoped access** — every company, campaign, and prospect is verified before any read or action
- **Revocable keys** — revoke anytime from [app.nuph.ai/extensions](https://app.nuph.ai/extensions)
- **Audit trail** — all API key usage is logged with `last_used_at`
- **No destructive actions** — MCP cannot delete data or modify account settings
- **Manual approval gate** — prospect approvals require user confirmation in the dashboard, preventing accidental outreach

See [SECURITY.md](SECURITY.md) for details and responsible disclosure.

## Protocol Details

| | |
|---|---|
| **Protocol** | [Model Context Protocol](https://modelcontextprotocol.io/) v2024-11-05 |
| **Transport** | Streamable HTTP (single POST endpoint) |
| **Format** | JSON-RPC 2.0 |
| **Endpoint** | `https://api.nuph.ai/functions/v1/outreach-mcp` |
| **Discovery** | `https://nuph.ai/.well-known/mcp.json` |
| **Auth** | `Authorization: Bearer <api_key>` (API key prefix: `outreach_`) |
| **Rate limits** | Based on your plan's credit balance |
| **Timeout** | 60 seconds per request |

See [docs/PROTOCOL.md](docs/PROTOCOL.md) for the full JSON-RPC reference.

## Documentation

- [Setup Guide](docs/SETUP.md) — Detailed setup for each MCP client
- [Tools Reference](docs/TOOLS.md) — All 9 tools with schemas
- [Protocol Reference](docs/PROTOCOL.md) — JSON-RPC 2.0 message format
- [Example Prompts](docs/EXAMPLES.md) — 50+ tested prompts by use case
- [Troubleshooting](docs/TROUBLESHOOTING.md) — Common issues and fixes
- [llms.txt](https://nuph.ai/llms.txt) — Full platform reference for LLMs

## Comparison with Other MCP Servers

If you use MCP for:
- **Linear, Jira, Asana** (project management) — nuph.ai is for LinkedIn outreach
- **GitHub** (code) — nuph.ai is for prospecting
- **Sentry** (errors) — nuph.ai is for sales pipeline
- **Stripe** (payments) — nuph.ai handles lead scoring and outreach

nuph.ai MCP is the first (and currently only) MCP server dedicated to **LinkedIn outreach, lead generation, and AI-powered sales prospecting**.

## Related

- [Official MCP Registry](https://registry.modelcontextprotocol.io/) — Discover other MCP servers
- [modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers) — Reference implementations
- [awesome-mcp-servers](https://github.com/wong2/awesome-mcp-servers) — Curated list
- [awesome-remote-mcp-servers](https://github.com/jaw9c/awesome-remote-mcp-servers) — Remote-only list

## Keywords

*LinkedIn MCP server, LinkedIn outreach AI, Claude Desktop LinkedIn, Cursor LinkedIn integration, B2B sales MCP, lead generation MCP, AI prospecting, Model Context Protocol LinkedIn, AI sales agent, LinkedIn automation Claude, AI-powered outreach, nuph.ai MCP, LinkedIn lead search Claude, AI lead scoring, Sales Navigator MCP, remote MCP server, streamable HTTP MCP, JSON-RPC MCP server, LinkedIn API Claude, AI LinkedIn prospecting, B2B outreach AI, sales automation MCP, Claude LinkedIn tools, MCP for recruiters, AI recruiting, LinkedIn agency tools, Chrome extension LinkedIn, AI-generated messages LinkedIn, personalized outreach AI, LinkedIn discovery tool*

## Contributing

This repo contains documentation, examples, and configuration — not the server source code. The server itself runs on nuph.ai infrastructure.

If you:
- Find a bug with the MCP server → [open an issue](https://github.com/nuph-ai/nuph-mcp-server/issues)
- Want to improve the docs → [submit a PR](https://github.com/nuph-ai/nuph-mcp-server/pulls)
- Have a tool request → [start a discussion](https://github.com/nuph-ai/nuph-mcp-server/discussions)

See [CONTRIBUTING.md](CONTRIBUTING.md) for details.

## Support

- **Email:** [here@nuph.ai](mailto:here@nuph.ai)
- **Documentation:** [nuph.ai/integrations/mcp/](https://nuph.ai/integrations/mcp/)
- **Help Center:** [nuph.ai/help-center/](https://nuph.ai/help-center/)
- **Status:** [nuph.ai/status/](https://nuph.ai/status/)

## License

MIT — See [LICENSE](LICENSE).

The MIT license applies to this repository (docs, examples, configs). The nuph.ai MCP server itself is proprietary and governed by the [nuph.ai Terms of Service](https://nuph.ai/terms/).

---

<div align="center">

**Built for the [Model Context Protocol](https://modelcontextprotocol.io/) by [nuph.ai](https://nuph.ai)**

If you like this, [star the repo](https://github.com/nuph-ai/nuph-mcp-server) and [give nuph.ai a try](https://nuph.ai) 🚀

</div>
