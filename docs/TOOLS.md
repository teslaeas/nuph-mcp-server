# Tools Reference

Complete reference for all 47 tools exposed by the nuph.ai MCP Server (v2.1.0).

All tools are authenticated with your API key. Access is scoped to data in companies you own.

**Two important contracts to know about every tool:**

1. **Write tools default to `dry_run: true`** — the first call never writes. It returns a structured preview (`preview`, `missing_required`, `missing_recommended`, `warnings`, `estimated_credits`, `ready_to_execute`, `next_call`). Pass `dry_run: false` to actually execute.
2. **`company_id` auto-resolves** when you own exactly one company. With > 1 you get a structured `ambiguous_company` error containing the option list. With 0 you get `no_company`.

---

## Discovery & Workspace

### `get_workspace_overview`

One-shot tour of the user's nuph.ai state. Best first call when starting a session.

**Input:** None

**Returns:** `{ companies, active_campaigns, credits_balance, recent_jobs, alerts }`. Each company/campaign is a small object suitable for the model to reason over without follow-up calls.

**Example:**
```
"Give me an overview of my workspace"
"What's the state of my nuph.ai account?"
```

---

### `suggest_next_actions`

Prioritised list of recommended next steps. Each action includes a `tool` name + suggested `args` + a `why` reason.

**Input:**
- `company_id` (string, optional) — limit to one company

**Returns:** `{ actions: [{ priority, tool, args, why }, ...] }`

Detects: high-score prospects pending approval, warm/hot prospects waiting > 24h, paused campaigns, low credits.

**Example:**
```
"What should I do today?"
"What's pending in my account?"
```

---

### `list_form_specs`

List every write action exposed by the MCP with brief metadata. Auto-discovery of all writable actions.

**Input:** None

**Returns:** Array of `{ id, target_table, notes, field_count, costs_credits }`.

**Example:**
```
"What write actions can the MCP do?"
```

---

### `get_form_spec`

Inspect the field spec for a single write action. Returns every field with help text, required/recommended flags, defaults, validation rules, and option references. **Always call this before any non-trivial write so you know what to ask the user for.**

**Input:**
- `id` (string, required) — Action id, e.g. `create_campaign`, `update_icp_profile`, `search_leads`

**Returns:** Full form spec with `fields[]` array.

**Example:**
```
"What fields do I need to provide to create a campaign?"
```

---

## Companies

### `list_companies`

List all companies owned by the authenticated user.

**Input:** None

**Returns:** Array of companies with `id`, `name`, `industry`, `description`, `website`, `linkedin_url`, `created_at`.

---

### `create_company`

Create a new company. **dry_run by default.**

**Input:**
- `name` (string, required)
- `industry`, `description`, `value_proposition`, `main_products_services`, `target_audience`, `website`, `linkedin_url`, `language` (all optional)
- `dry_run` (boolean, default `true`)

**Returns:** Preview if `dry_run` (default), or `{ result, credits_consumed: 0, hints }` after execute.

---

### `update_company`

Edit company details. **dry_run by default.**

**Input:**
- `company_id` (string, required)
- Any field from `create_company` (all optional)
- `dry_run` (boolean, default `true`)

---

### `delete_company`

Delete company + all campaigns / prospects (destructive). **dry_run by default; `confirm: true` always required even with `dry_run: false`.**

**Input:**
- `company_id` (string, required)
- `confirm` (boolean, required — must be `true`)
- `dry_run` (boolean, default `true`)

---

### `scrape_company_website`

Scrape a website URL and extract company data (name, industry, description, specialties, size, HQ). **Costs credits. dry_run by default.**

**Input:**
- `website` (string, required)
- `company_name`, `linkedin_url` (optional)
- `dry_run` (boolean, default `true`)

---

### `generate_company_profile`

Use AI to generate description, target audience, value prop, products from basic company info. **Costs credits. dry_run by default.**

---

## Campaigns

### `list_campaigns`

List campaigns for a company with cursor pagination + exact total count. Auto-resolves `company_id`.

**Input:**
- `company_id` (string, optional — auto-resolved if 1 company)
- `status` (string, optional) — `"active" | "paused" | "archived" | "completed"`
- `limit` (number, default 50, max 200)
- `cursor` (string, optional — from previous response)

**Returns:** `{ campaigns, total, returned, has_more, next_cursor, resolved_context }`

---

### `get_campaign`

Get detailed campaign info including ICP profile and prospect stats.

**Input:**
- `campaign_id` (string, required)

**Returns:** Campaign object with full ICP profile + `stats: { total_prospects, approved }`.

---

### `create_campaign`

Create a campaign. **dry_run by default.**

**Input:**
- `company_id` (string, required)
- `name` (string, required)
- `description`, `language`, `icp_profile_id`, `sender_context`, `tone_id`, `mode_id` (all optional)
- `dry_run` (boolean, default `true`)

---

### `update_campaign`, `delete_campaign`

Standard update/delete with dry_run. `delete_campaign` requires `confirm: true`.

---

### `enhance_campaign`

Use AI to generate/improve a campaign description and suggest LinkedIn search keywords. **Costs credits. dry_run by default.**

---

### `get_campaign_stats`

Exact prospect counts by status for a single campaign.

**Input:** `campaign_id` (string, required)

**Returns:** `{ campaign_id, total, by_status: { new, approved, connection_sent, connected, replied, ... } }`

---

### `get_company_stats`

Prospect counts across all campaigns in a company. Auto-resolves `company_id`.

---

## ICP Profiles

### `list_icp_profiles`

List all ICP profiles for a company. Auto-resolves `company_id`.

---

### `get_icp_profile`

**Input:** `icp_profile_id` (string, required)

---

### `create_icp_profile`, `update_icp_profile`

Standard create/update with dry_run. Fields: `name`, `target_titles`, `target_locations`, `target_industries`, `company_size_min/max`, `target_seniority`, `target_functions`, `target_company_headcount`, `exclude_keywords`.

---

## Prospects (smart search + intent)

### `query_prospects` — RECOMMENDED

**Unified prospect search/list/export.** Cursor pagination, exact total count, smart pitch_match scoring (pure SQL, no AI cost). Replaces `list_prospects`, `search_prospects`, `export_prospects` (which remain as deprecated wrappers).

**Input:**
- `company_id` (string, optional — auto-resolved)
- `campaign_id` (string, optional)
- `search_text` (string, optional) — ILIKE on first_name|last_name|current_title|current_company|email_enriched
- `pitch_match` (string, optional) — Free-text pitch; prospects ranked by keyword match on title and company plus their existing ICP score. Pure SQL — costs no AI credits.
- `filters` (object, optional):
  - `status` (string or array) — `"new" | "approved" | "connected" | "warm" | "hot" | "converted" | ...`
  - `min_score`, `max_score` (number, 0-10)
  - `country` (array of strings)
  - `city` (array of strings)
  - `title_contains` (array of strings)
  - `company_contains` (array of strings)
  - `has_email` (boolean)
  - `enriched` (boolean)
  - `starred` (boolean)
  - `min_connections` (number)
  - `replied` (boolean) — `true` = warm/hot/converted
  - `created_after`, `created_before` (ISO 8601)
- `sort_by` (`"score" | "recency" | "name"`, default `score`)
- `sort_dir` (`"asc" | "desc"`, default `desc`)
- `limit` (number, default 50, max 500)
- `cursor` (string, optional)

**Returns:** `{ prospects, total, returned, has_more, next_cursor, applied_filters, resolved_context, tip }`. Each prospect includes `id`, `first_name`, `last_name`, `current_title`, `current_company`, `location`, `country`, `city`, `email_enriched`, **`linkedin_url`**, `icp_score`, `status`, `enriched`, `connections_count`, `created_at`, `match_score`, `match_reasoning`.

**Example:**
```
"Show me ALL my connected prospects with score >= 7"
"Search for prospects in Spain or Germany with title containing 'CTO' or 'VP'"
"Find prospects who replied in the last 7 days"
```

---

### `find_pitchable_prospects`

Intent tool: pass a value-proposition text → returns ranked list with `match_score` + `match_reasoning`. Defaults to connected/warm/hot/replied prospects with score≥6, sorted by combined match + ICP score. **No AI credits consumed.**

**Input:**
- `pitch` (string, required) — Value proposition or product description (free text)
- `company_id` (string, optional — auto-resolved)
- `statuses` (array, optional) — override default `["connected","warm","hot","replied"]`
- `min_score` (number, optional — default 6)
- `limit` (number, optional — default 25)

**Returns:** Same shape as `query_prospects`, but pre-configured for pitch matching.

**Example:**
```
"Find prospects best fit for AI outreach automation for SaaS sales"
"Who can I pitch our marketing analytics tool to?"
```

---

### `get_prospect`

Get full prospect details: profile, experience, education, skills, languages, generated messages, score breakdown.

**Input:** `prospect_id` (string, required)

**Returns:** Complete prospect object + `generated_messages` array.

---

### `update_prospect`

Update notes, deal value, status, starred. **dry_run by default.** Status changes that require the extension (`connection_sent`, `msg1_sent`, etc.) are blocked.

---

### `list_prospects` / `search_prospects` / `export_prospects` (DEPRECATED)

Thin wrappers around `query_prospects` for backward compatibility. The original "max 50" cap is gone (they now delegate to `query_prospects` which paginates up to 500). Use `query_prospects` directly for new code.

---

## Lead Search & Enrichment & Jobs

### `search_leads`

Launch a new LinkedIn search. Runs asynchronously — results appear in your prospects list in 1-3 minutes. **Costs credits. dry_run by default — preview shows `estimated_credits`.**

**Input:**
- `campaign_id` (string, required)
- `keywords` (array of strings, required) — Job titles, e.g. `["CTO", "VP Engineering"]`
- `locations` (array of strings, required) — Countries/cities, e.g. `["Spain", "Germany"]`
- `results_per_keyword` (number, optional) — Min 25, default 25
- `seniority_levels` (array of strings, optional) — IDs `100`–`400`
- `functions` (array of strings, optional) — IDs `1`–`26`
- `company_headcount` (array of strings, optional) — `"A"` (Self) to `"I"` (10K+)
- `dry_run` (boolean, default `true`)

**Seniority IDs:** `100` In Training, `110` Entry, `120` Senior, `130` Strategic, `200` Entry Manager, `210` Experienced Manager, `300` Director, `310` VP, `320` CXO, `400` Owner

**Function IDs:** `1` Accounting, `2` Admin, `3` Arts, `4` Biz Dev, `5` Community, `6` Consulting, `7` Education, `8` Engineering, `9` Entrepreneurship, `10` Finance, `11` Healthcare, `12` HR, `13` IT, `14` Legal, `15` Marketing, `16` Media, `17` Military, `18` Operations, `19` Product Mgmt, `20` Program, `21` Purchasing, `22` QA, `23` Real Estate, `24` Research, `25` Sales, `26` Support

**Headcount:** `A` Self, `B` 1-10, `C` 11-50, `D` 51-200, `E` 201-500, `F` 501-1K, `G` 1K-5K, `H` 5K-10K, `I` 10K+

**Returns:** Preview (with estimated_credits) when `dry_run`. After execute: `{ success, message, job_id }`.

---

### `enrich_prospects`

Enrich specific prospects with full LinkedIn data. **Costs credits per profile (cached profiles free). dry_run by default.**

**Input:** `prospect_ids` (array of strings, required), `dry_run` (default `true`)

---

### `check_enrichment_status`

Check status of a specific job by ID.

**Input:** `job_id` (string, required)

---

### `list_recent_jobs`

List recent scraping/enrichment jobs with cursor pagination. Auto-resolves `company_id`. Auto-tip when active jobs in flight ("poll every 30-60s").

**Input:**
- `company_id` (string, optional)
- `status` (string, optional) — `"pending" | "running" | "processing" | "completed" | "failed" | "cancelled"`
- `job_type` (string, optional) — `"search" | "enrichment"`
- `limit` (number, default 20, max 100)
- `cursor` (string, optional)

---

### `cancel_job`

Cancel a queued (status=pending) scraping job and refund reserved credits. **dry_run by default.**

**Input:** `job_id` (string, required), `dry_run` (default `true`)

Cannot cancel jobs already running or completed.

---

## Messages

### `list_messages`

List all generated messages for a prospect (steps 0-3, all versions). Cursor pagination + exact total.

**Input:** `prospect_id` (string, required), `limit` (default 50, max 200), `cursor` (optional)

---

### `generate_message`

Generate or regenerate an AI message for a prospect. **Does NOT approve or send. Costs credits. dry_run by default.**

**Input:**
- `prospect_id` (string, required)
- `step` (number, optional) — `0` connection note, `1` follow-up, `2` value, `3` close. Default `0`.
- `dry_run` (boolean, default `true`)

---

### `update_message`

Edit a generated message body. **The message won't be sent — sending requires the Chrome extension.**

When passing only `message_id` + `new_body` (no other fields besides `dry_run`), this acts as a simple body edit and skips the dry_run preview.

**Input:** `message_id` (string, required), `new_body` (string, required)

---

### `generate_chat_reply`

AI reply for ongoing LinkedIn chat. **Costs credits. dry_run by default.**

**Input:** `prospect_id`, `tone_id?`, `mode_id?`, `target_language?`, `dry_run`

---

## LinkedIn Tools

### `audit_linkedin_profile`

Start a LinkedIn profile audit. Returns a `run_id` to poll with `check_linkedin_audit`. **Costs credits. dry_run by default.**

**Input:** `linkedin_url` (string, required), `dry_run` (default `true`)

---

### `check_linkedin_audit`

Poll a LinkedIn audit run until it succeeds, returning profile data.

**Input:** `run_id` (string, required)

---

## Analytics & Activity

### `get_dashboard_metrics`

Funnel metrics for a company: total prospects, approved, connected, replied, converted, conversion rates, avg ICP score. Auto-resolves `company_id`.

**Input:** `company_id` (optional), `campaign_id` (optional)

---

### `get_top_locations`

Geographic distribution of prospects (country or city).

**Input:** `company_id` (optional), `group_by` (`"country" | "city"`), `limit` (default 20)

---

### `list_recent_activity`

Activity log for a company with cursor pagination + exact total. **Includes every MCP call** (`source='mcp'`) so users can audit AI activity from the dashboard or via the MCP itself.

**Input:** `company_id` (optional), `limit` (default 50, max 200), `cursor` (optional)

---

## Credits & Config

### `get_credits_balance`

**Input:** None
**Returns:** `{ balance, plan, name }`

---

### `get_credit_transactions`

Detailed credit transaction history with cursor pagination + exact total.

**Input:** `limit` (default 50, max 200), `cursor` (optional), `type` (optional — filter by `scraping`, `enrichment`, `ai_chat`, `ai_message`, `refund`, `purchase`)

---

### `list_writing_styles`

Available tones, modes, constraints for AI message generation.

---

### `get_user_settings`

User's default AI config (tone, mode, language, ICP strictness).

---

## What the MCP server CANNOT do

- ❌ **Approve prospects** — Manual user decision via dashboard at [app.nuph.ai](https://app.nuph.ai)
- ❌ **Send LinkedIn messages or connection requests** — Requires the official [Chrome Extension](https://chromewebstore.google.com/detail/nuphai-linkedin-outreach/ekcjniemnbdjjobajommjdnoimhdijel) installed in the browser where LinkedIn is logged in
- ❌ **Modify account settings, billing, or plan**

This is **intentional**. The MCP server is designed for full pipeline management (search, create, edit, analyze, preview) but the final outreach actions stay user-gated for safety.

## Credit costs

Tools that cost credits:
- `search_leads` — per-result cost (preview shows `estimated_credits`)
- `enrich_prospects` — per-profile cost (cached profiles free)
- `generate_message` — ~1 credit per generation
- `generate_chat_reply` — ~1 credit per reply
- `audit_linkedin_profile` — fixed per-audit cost
- `scrape_company_website`, `generate_company_profile`, `enhance_campaign` — fixed per-call cost

All other tools are free (read-only).

See [nuph.ai/pricing](https://nuph.ai/pricing/) for current plans.

## Rate limits

- 60 calls/min per API key
- 600 calls/hour per API key

When exceeded, the server returns HTTP 429 with a structured `rate_limited` error including `retry_after_seconds`.

Configurable per-account by editing `outreach_ai_config.mcp_max_calls_per_min` and `mcp_max_calls_per_hour`.
