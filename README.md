<div align="center">

# nuph.ai MCP Server

### Control your LinkedIn outreach from Claude, Cursor, or any MCP-compatible AI

[![MCP](https://img.shields.io/badge/Model%20Context%20Protocol-v2025--06--18-F5A623?style=for-the-badge)](https://modelcontextprotocol.io/)
[![Streamable HTTP](https://img.shields.io/badge/Transport-Streamable%20HTTP-CF7153?style=for-the-badge)](https://modelcontextprotocol.io/specification/2025-06-18/basic/transports)
[![Server](https://img.shields.io/badge/Server-v2.1.0-10B981?style=for-the-badge)](https://nuph.ai/.well-known/mcp.json)
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
- **Smart pitch matching** — "Find the prospects best fit for AI outreach automation" (pure SQL ranking, no AI cost)
- **Conversational by default** — every write tool previews before executing; no surprise creates or credit spends
- **Auto-resolves your company** — if you own one, you don't need to specify; if many, the model knows to ask
- **Query your pipeline** — "Show me ALL my connected prospects" (returns the real total, not capped at 50)
- **Workspace tour in 1 call** — `get_workspace_overview` returns companies, campaigns, credits, recent jobs, alerts
- **Self-discovers what to ask** — `list_form_specs` + `get_form_spec` let the model know exactly which fields each action needs
- **Zero setup** — no local install, just paste a URL + API key into your MCP client

## What's new in v2.1.0 (May 2026)

- **47 tools** (was 38) — see the [Tools section](#47-tools-available) below
- **`query_prospects`** — unified smart search; returns up to 500/page with cursor pagination + exact total. Replaces `list_prospects` / `search_prospects` / `export_prospects` (still callable as deprecated wrappers).
- **`find_pitchable_prospects`** — ranks prospects against a free-text pitch using pure SQL (no AI credits).
- **`get_workspace_overview`, `suggest_next_actions`** — high-level intent tools for orientation.
- **`list_form_specs`, `get_form_spec`** — auto-discovery of every write action's required/recommended fields, defaults and option references.
- **`list_recent_jobs`, `cancel_job`** — monitor and cancel pending scraping/enrichment jobs.
- **`dry_run` by default on every write** — first call returns a structured preview (`missing_required`, `missing_recommended`, `warnings`, `estimated_credits`, `ready_to_execute`, `next_call`); only with `dry_run: false` does it execute.
- **Auto-resolution of `company_id`** — if you own exactly one company, you don't need to pass it. If many, the response is a structured `ambiguous_company` error with the option list + a `retry_with` hint.
- **Cursor pagination + exact totals** on every list tool — the model finally knows how many records exist.
- **Rate limiting** — 60 calls/min, 600/hour per API key (returns HTTP 429 with `retry_after_seconds`).
- **Audit log** — every MCP call is recorded in `outreach_activity_log` with `source='mcp'` so users can see what the AI did from their dashboard.
- **Protocol upgraded** to MCP v2025-06-18.

## Quick Start

### 1. Get your API key

Sign in at [app.nuph.ai/extensions](https://app.nuph.ai/extensions) and create a new API key. Copy it (it starts with `outreach_`).

Don't have a nuph.ai account? [Sign up free](https://nuph.ai) — 50 credits/month, no credit card.

### 2. Connect your AI client

#### Claude.ai (web — claude.ai)

Claude.ai does not support custom Authorization headers. Use the API key as a URL query parameter:

1. Open [claude.ai](https://claude.ai) → **Settings → Connectors → Add custom connector**
2. **Name:** `nuph.ai`
3. **URL** (replace `YOUR_API_KEY` with your actual key):
   ```
   https://api.nuph.ai/functions/v1/outreach-mcp?apiKey=YOUR_API_KEY
   ```
4. Save. The 47 tools appear immediately.

> The server accepts multiple query param names: `apiKey`, `apikey`, `api_key`, `key`, `token`.

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
You: Give me an overview of my workspace

Claude:
  → get_workspace_overview()
  ✓ 3 companies, 25 active campaigns, 1463 credits, 5 recent jobs

You: Show me ALL my connected prospects

Claude:
  → query_prospects(filters: { status: "connected" }, limit: 500)
  ✓ Returns 295 prospects (total: 295, has_more: false)
    — every linkedin_url included so you can open profiles

You: Find me prospects best fit for "AI outreach automation for SaaS sales"

Claude:
  → find_pitchable_prospects(pitch: "AI outreach automation for SaaS sales")
  ✓ 82 ranked matches, top: Priyank Goel (Salesforce Consultant) — match_score: 80
    reasoning: ["Title matches keyword"]

You: Create a campaign for Q3 in Spain

Claude:
  → create_campaign(name: "Q3 Spain")
  ✓ Returns preview with missing_recommended fields and ready_to_execute=true.
    Asks: "Which ICP profile should I link, and which tone?"

You: Pro tone, link the 'Marketing Leads EU' ICP

Claude:
  → create_campaign(..., dry_run: false)
  ✓ Campaign created. credits_consumed: 0.
```

## 47 Tools Available

Full platform coverage. You can create, read, update, delete everything **except** approve prospects and send messages (those require the dashboard + Chrome extension for user safety).

### Discovery & Workspace
| Tool | Description |
|------|-------------|
| `get_workspace_overview` | One-shot tour: companies, active campaigns, credits, recent jobs, alerts. Best first call. |
| `suggest_next_actions` | Prioritised list of recommended next steps (pending approvals, unanswered replies, low credits) with pre-computed tool + args. |
| `list_form_specs` | Every write action exposed by the MCP with brief metadata. Use to discover what's possible. |
| `get_form_spec` | Field spec for a single action (help text, required/recommended flags, defaults, option refs). Always call before non-trivial writes. |

### Companies
| Tool | Description |
|------|-------------|
| `list_companies` | List all companies you own |
| `create_company` | Create a new company (dry_run by default) |
| `update_company` | Edit company details (dry_run by default) |
| `delete_company` | Delete company + all campaigns/prospects (destructive; needs `confirm: true`) |
| `scrape_company_website` | Auto-extract company data from a website URL (costs credits) |
| `generate_company_profile` | AI generates description, value prop, target audience (costs credits) |

### Campaigns
| Tool | Description |
|------|-------------|
| `list_campaigns` | List campaigns with cursor pagination + exact total. Auto-resolves company_id. |
| `get_campaign` | Full campaign details + stats |
| `create_campaign` | Create campaign (dry_run by default) |
| `update_campaign` | Edit campaign (dry_run by default) |
| `delete_campaign` | Delete campaign + prospects (destructive; needs `confirm: true`) |
| `enhance_campaign` | AI improves description + suggests keywords (costs credits) |
| `get_campaign_stats` | Exact prospect counts by status |
| `get_company_stats` | Prospect counts across all campaigns in a company |

### ICP Profiles
| Tool | Description |
|------|-------------|
| `list_icp_profiles` | List ICPs for a company (auto-resolves company_id) |
| `get_icp_profile` | Full ICP details |
| `create_icp_profile` | Create targeting criteria (dry_run by default) |
| `update_icp_profile` | Edit ICP (dry_run by default) |

### Prospects (smart search + intent)
| Tool | Description |
|------|-------------|
| `query_prospects` | **Unified prospect search/list/export.** Up to 500/page with cursor pagination + exact total. Filter by status, score range, country, city, title/company contains, has_email, replied, dates. Pass `pitch_match` for ranked smart search. Auto-resolves company_id. |
| `find_pitchable_prospects` | Intent tool: pass a value-proposition text → returns ranked list with `match_score` + `match_reasoning`. Defaults to connected/warm/hot/replied with score≥6. No AI credits. |
| `get_prospect` | Full prospect details + messages + score breakdown |
| `update_prospect` | Edit notes, deal value, starred, status (dry_run by default; non-extension statuses only) |
| `list_prospects` / `search_prospects` / `export_prospects` | **DEPRECATED.** Wrappers around `query_prospects` for back-compat. Use `query_prospects` for new code. |

### Lead Search & Enrichment & Jobs
| Tool | Description |
|------|-------------|
| `search_leads` | Launch LinkedIn search with advanced filters (costs credits; preview shows estimated_credits) |
| `enrich_prospects` | Enrich specific prospects with full LinkedIn data (costs credits per profile; cached profiles free) |
| `check_enrichment_status` | Check status of a specific job by ID |
| `list_recent_jobs` | List recent scraping/enrichment jobs with cursor pagination. Auto-tip when active jobs in flight. |
| `cancel_job` | Cancel a queued (status=pending) job and refund reserved credits (dry_run by default) |

### Messages
| Tool | Description |
|------|-------------|
| `list_messages` | All generated messages for a prospect (cursor pagination + total) |
| `generate_message` | Generate AI message for step 0–3 (costs credits; does NOT send) |
| `update_message` | Edit message body (simple body-only edits skip dry_run preview) |
| `generate_chat_reply` | AI reply for ongoing LinkedIn chat (costs credits) |

### LinkedIn Tools
| Tool | Description |
|------|-------------|
| `audit_linkedin_profile` | Start audit of any public LinkedIn profile (costs credits) |
| `check_linkedin_audit` | Poll audit status |

### Analytics & Activity
| Tool | Description |
|------|-------------|
| `get_dashboard_metrics` | Funnel metrics: total, approved, connected, replied, conversion rates |
| `get_top_locations` | Geographic distribution of prospects (country or city) |
| `list_recent_activity` | Activity log with cursor pagination. Includes every MCP call (`source='mcp'`) so users can audit AI activity. |

### Credits & Config
| Tool | Description |
|------|-------------|
| `get_credits_balance` | Current balance + plan |
| `get_credit_transactions` | Detailed transaction history (cursor pagination + total) |
| `list_writing_styles` | Available tones/modes/constraints for AI messages |
| `get_user_settings` | User's default AI config |

### What this MCP does NOT do (by design)
- ❌ **Cannot approve prospects** — approval requires dashboard review at [app.nuph.ai/prospects](https://app.nuph.ai/prospects)
- ❌ **Cannot send LinkedIn messages or connection requests** — requires the official [Chrome Extension](https://chromewebstore.google.com/detail/nuphai-linkedin-outreach/ekcjniemnbdjjobajommjdnoimhdijel) installed where LinkedIn is logged in
- ❌ **Cannot modify billing, plan, or purchase credits** — dashboard-only

## The `dry_run` contract (preview-then-confirm)

Every write tool defaults to `dry_run: true`. The first call **never writes** — it returns a structured preview:

```json
{
  "preview": {
    "name": "Pilot Q2",
    "language": "en",                          // resolved from company.language
    "tone_id": "balanced"                      // resolved from user_settings
  },
  "resolved_defaults": [
    {"field": "language", "value": "en", "source": "company.language"}
  ],
  "missing_required": [],
  "missing_recommended": [
    { "field": "icp_profile_id", "type": "id_ref", "options_ref": "list_icp_profiles", "help": "Link to an existing ICP profile" }
  ],
  "warnings": [],
  "estimated_credits": null,
  "preflight": { "checks_passed": [], "checks_failed": [] },
  "ready_to_execute": true,
  "next_call": {
    "tool": "create_campaign",
    "arguments": { /* echo with defaults */ },
    "with": { "dry_run": false }
  }
}
```

The model uses `missing_recommended` to ask the user smart questions (in any language — the help text is translated client-side from English). On confirm, it calls again with `dry_run: false`.

## Auto-resolution of `company_id`

Tools that take `company_id` are smart about it:

- **You own 1 company** → auto-resolved; the response includes `resolved_context: { company_id, auto_resolved: true }`.
- **You own 0 companies** → structured `no_company` error with `help: "Create one first with create_company"`.
- **You own >1 companies** → structured `ambiguous_company` error with the full list of `{id, name}` and a `retry_with: { field: "company_id", from: "options[].id" }` hint.

Same pattern for `campaign_id` where applicable.

## Smart search with `query_prospects`

Replaces `list_prospects` / `search_prospects` / `export_prospects`. Cursor pagination, exact total, rich filters, optional pitch ranking:

```json
{
  "company_id": "<auto-resolved if 1 company>",
  "campaign_id": "<optional>",
  "search_text": "<optional ILIKE on name/title/company/email>",
  "pitch_match": "<optional free text — ranks results>",
  "filters": {
    "status": ["connected", "warm", "hot"],
    "min_score": 7,
    "country": ["Spain", "Germany"],
    "title_contains": ["VP", "CTO"],
    "has_email": true,
    "replied": false,
    "min_connections": 500
  },
  "sort_by": "score | recency | name",
  "limit": 500,
  "cursor": "<from previous response>"
}
```

Response includes `total`, `returned`, `has_more`, `next_cursor`, `applied_filters`, `resolved_context`, and a `tip` string for navigation.

## Example Prompts

Try these in your connected AI client:

- *"Give me an overview of my nuph.ai workspace"*
- *"What should I do today?"* (calls `suggest_next_actions`)
- *"Show me ALL my connected prospects in [company]"*
- *"Find prospects best fit for AI outreach automation for SaaS sales teams"*
- *"Search for 25 VP Engineering in Germany with company size 51-200"*
- *"Create a campaign called Q3 Pilot in Spain"* (returns preview, asks ICP/tone)
- *"How many credits do I have left?"*
- *"Show me my running scraping jobs"*
- *"Generate a preview connection message for prospect [ID]"*
- *"Which of my campaigns has the most connected prospects?"*
- *"Find 50 CTOs in Spain who speak Spanish"*

See [docs/EXAMPLES.md](docs/EXAMPLES.md) for more.

## What is nuph.ai?

[nuph.ai](https://nuph.ai) is an AI-powered LinkedIn outreach platform for B2B sales teams, recruiters, and agencies. It automates the full outreach workflow:

1. **Discover** — Search LinkedIn with advanced filters
2. **Enrich** — Get verified emails + 50+ data points per profile
3. **Score** — AI scores every prospect 0-10 against your ICP
4. **Generate** — AI writes personalized connection notes and follow-up sequences
5. **Send** — Chrome extension sends connections directly from LinkedIn

**Plans:** Free ($0, 50 credits), Starter ($59, 1,000 credits), Pro ($129, 2,500 credits), Agency ($299, 5,000 credits).

## Security

- **API key authentication** on every request (Bearer token or `?apikey=` query param)
- **Ownership verification** — you can only access data in companies you own
- **Scoped access** — every company, campaign, and prospect is verified before any read or action
- **Rate limited** — 60 calls/min, 600 calls/hour per API key (returns HTTP 429 with `retry_after_seconds`)
- **Audit trail** — every MCP call is logged to `outreach_activity_log` with `source='mcp'` and visible in `list_recent_activity`
- **Revocable keys** — revoke anytime from [app.nuph.ai/extensions](https://app.nuph.ai/extensions)
- **Manual approval gate** — prospect approvals require user confirmation in the dashboard, preventing accidental outreach
- **Preview-then-confirm** — every write requires a confirmation roundtrip via `dry_run`

See [SECURITY.md](SECURITY.md) for details and responsible disclosure.

## Protocol Details

| | |
|---|---|
| **Server version** | v2.1.0 |
| **Protocol** | [Model Context Protocol](https://modelcontextprotocol.io/) v2025-06-18 |
| **Transport** | Streamable HTTP (single POST endpoint) |
| **Format** | JSON-RPC 2.0 |
| **Endpoint** | `https://api.nuph.ai/functions/v1/outreach-mcp` |
| **Discovery** | `https://nuph.ai/.well-known/mcp.json` |
| **Auth** | `Authorization: Bearer <api_key>` (API key prefix: `outreach_`) |
| **Rate limit** | 60 calls/min, 600 calls/hour per API key |
| **Timeout** | 60 seconds per request |

See [docs/PROTOCOL.md](docs/PROTOCOL.md) for the full JSON-RPC reference.

## Documentation

- [Setup Guide](docs/SETUP.md) — Detailed setup for each MCP client
- [Tools Reference](docs/TOOLS.md) — All 47 tools with schemas
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

*LinkedIn MCP server, LinkedIn outreach AI, Claude Desktop LinkedIn, Cursor LinkedIn integration, B2B sales MCP, lead generation MCP, AI prospecting, Model Context Protocol LinkedIn, AI sales agent, LinkedIn automation Claude, AI-powered outreach, nuph.ai MCP, LinkedIn lead search Claude, AI lead scoring, Sales Navigator MCP, remote MCP server, streamable HTTP MCP, JSON-RPC MCP server, LinkedIn API Claude, AI LinkedIn prospecting, B2B outreach AI, sales automation MCP, Claude LinkedIn tools, MCP for recruiters, AI recruiting, LinkedIn agency tools, Chrome extension LinkedIn, AI-generated messages LinkedIn, personalized outreach AI, LinkedIn discovery tool, MCP dry_run, MCP preview-then-confirm, conversational MCP, smart prospect search, pitch_match MCP*

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
