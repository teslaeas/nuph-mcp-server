# Security Policy

## Supported Versions

The nuph.ai MCP Server is a hosted service at `https://api.nuph.ai/functions/v1/outreach-mcp`. We always run the latest version — there are no versioned releases to support.

## Reporting a Vulnerability

If you discover a security vulnerability in the nuph.ai MCP Server, please report it responsibly:

**Email:** [security@nuph.ai](mailto:security@nuph.ai) (or [here@nuph.ai](mailto:here@nuph.ai) if security@ doesn't respond)

**Please include:**
- A clear description of the issue
- Steps to reproduce
- Potential impact
- Any proof-of-concept code (if applicable)

**What to expect:**
- We'll acknowledge your report within 48 hours
- We'll investigate and keep you updated on progress
- We'll credit you in any public disclosure (with your permission)

## Security Measures

The nuph.ai MCP Server implements several security layers:

### Authentication
- Every request requires an API key via `Authorization: Bearer <key>` header (or `?apikey=` query param for clients without custom-header support)
- API keys use `crypto.getRandomValues()` (not Math.random)
- Keys are prefixed with `outreach_` for easy identification
- Keys can be revoked at any time from [app.nuph.ai/extensions](https://app.nuph.ai/extensions)
- All key usage is logged with `last_used_at` timestamp

### Authorization
- Every tool call verifies ownership before returning data
- You can only access companies, campaigns, and prospects you own
- Cross-user access is blocked at every endpoint
- Auto-resolution of `company_id` returns a structured `ambiguous_company` error with the option list when you own more than one company — never silently picks the wrong one

### Rate limiting (per API key)
- 60 calls/minute, 600 calls/hour (configurable via `outreach_ai_config.mcp_max_calls_per_min`)
- Returns HTTP 429 with `Retry-After` header and structured `rate_limited` error
- Independent of credit balance — protects the platform from runaway loops or leaked keys

### Audit log
- Every MCP call is recorded in `outreach_activity_log` with `source='mcp'`
- Includes: tool name, sha256 of arguments, dry_run flag, duration_ms, credits_consumed, is_error, error_code, api_key_id, user_agent
- Visible from the dashboard or via the `list_recent_activity` MCP tool — users can see exactly what the AI did on their account

### Data Protection
- Row-Level Security (RLS) on all database tables
- No service-role keys exposed to clients
- Internal edge function calls use service-role + impersonation header (`x-impersonate-user-id`) — no magic-link emails sent on every call (was a side effect of the v1 implementation, removed in v2.1.0)
- All data in transit uses HTTPS (TLS 1.3)
- No data is sold or shared with third parties
- GDPR compliant — users can delete their account and all data anytime

### Action Safety — preview-then-confirm
- **Every write tool defaults to `dry_run: true`** — the first call always returns a structured preview (`missing_required`, `missing_recommended`, `warnings`, `estimated_credits`, `ready_to_execute`) instead of executing. The model only writes after explicitly passing `dry_run: false`.
- **Destructive actions** (`delete_company`, `delete_campaign`, `cancel_job`) require `confirm: true` on top of `dry_run: false`.
- **Status changes that require the Chrome extension** (`connection_sent`, `msg1_sent`, etc.) are blocked at the API level — you cannot fake an extension-only state via the MCP.
- The MCP server is intentionally limited in what it can do:
  - ❌ Cannot approve prospects (manual dashboard action required)
  - ❌ Cannot send LinkedIn messages or connection requests (Chrome extension required)
  - ❌ Cannot modify billing or account settings
- ✅ Can: launch searches (you pay), generate message previews (you pay), create/update/delete companies/campaigns/ICPs/prospects (with dry_run + confirm), audit LinkedIn profiles (you pay), scrape company info (you pay)

### Chrome Extension Safety
- The official Chrome extension validates that the logged-in LinkedIn account matches the configured nuph.ai company before sending any messages
- This prevents accidental outreach from the wrong LinkedIn account
- Per-device detection — installing on one browser doesn't expose other browsers

## What to Do If Your API Key is Compromised

1. Immediately revoke the key at [app.nuph.ai/extensions](https://app.nuph.ai/extensions)
2. Create a new key and update your MCP client configuration
3. Review recent activity in your dashboard
4. Email [security@nuph.ai](mailto:security@nuph.ai) if you suspect data was accessed

## Out of Scope

The following are not considered vulnerabilities:
- Missing security headers on public pages (we follow industry standards)
- Reports about DDoS — please don't test this
- Social engineering of nuph.ai staff
- Issues in third-party services (report to them directly)
- Rate limiting (it's intentional)

## Hall of Fame

Responsible disclosure researchers will be credited here (with permission).

---

Thank you for helping keep nuph.ai and our users safe.
