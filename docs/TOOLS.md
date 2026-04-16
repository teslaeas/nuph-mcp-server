# Tools Reference

Complete reference for all 38 tools exposed by the nuph.ai MCP Server.

All tools are authenticated with your API key. Access is scoped to data in companies you own.

---

## `list_companies`

List all companies owned by the authenticated user.

**Input:** None

**Returns:** Array of companies with `id`, `name`, `industry`, `description`, `website`, `linkedin_url`, `created_at`.

**Example:**
```
"List all my companies"
```

---

## `get_credits_balance`

Get the user's current credit balance and plan info.

**Input:** None

**Returns:** `{ balance: number, plan: string, name: string }`

**Example:**
```
"How many credits do I have left?"
"What plan am I on?"
```

---

## `list_campaigns`

List campaigns for a company.

**Input:**
- `company_id` (string, required) — Company ID
- `status` (string, optional) — `"active"` | `"paused"` | `"archived"`

**Returns:** Array of campaigns with `id`, `name`, `status`, `language`, `created_at`. Up to 50.

**Example:**
```
"Show me my active campaigns for company [name]"
"List all paused campaigns"
```

---

## `get_campaign`

Get detailed campaign info including ICP profile and prospect stats.

**Input:**
- `campaign_id` (string, required)

**Returns:** Campaign object with full ICP profile + `stats: { total_prospects, approved }`.

**Example:**
```
"Get stats for the [campaign name] campaign"
"Show me the ICP profile for my top campaign"
```

---

## `list_prospects`

List prospects with optional filters.

**Input:**
- `company_id` (string, required)
- `campaign_id` (string, optional) — Filter by campaign
- `status` (string, optional) — `"new"`, `"approved"`, `"connection_sent"`, `"connected"`, etc.
- `min_score` (number, optional) — Minimum ICP score (0-10)
- `limit` (number, optional) — Max results (default 25, max 50)

**Returns:** Array of prospects with key fields: `id`, `first_name`, `last_name`, `current_title`, `current_company`, `location`, `country`, `email_enriched`, `icp_score`, `status`, `connections_count`, `created_at`.

**Sorted by:** ICP score descending.

**Example:**
```
"Show me my top 10 prospects with score above 8"
"List all new prospects from the Germany campaign"
"Find connected prospects with score 9+"
```

---

## `get_prospect`

Get full prospect details: profile, experience, education, skills, languages, generated messages, score breakdown.

**Input:**
- `prospect_id` (string, required)

**Returns:** Complete prospect object + `generated_messages` array with all AI-generated message versions.

**Example:**
```
"Tell me everything about prospect [ID]"
"What messages have been generated for [name]?"
```

---

## `search_leads`

Launch a new LinkedIn search. Runs asynchronously — results appear in your prospects list in 1-3 minutes. **Costs credits.**

**Input:**
- `campaign_id` (string, required)
- `keywords` (array of strings, required) — Job titles, e.g. `["CTO", "VP Engineering"]`
- `locations` (array of strings, required) — Countries/cities, e.g. `["Spain", "Germany"]`
- `results_per_keyword` (number, optional) — Min 25, default 25
- `seniority_levels` (array of strings, optional) — IDs: `100` (In Training) to `400` (Owner)
- `functions` (array of strings, optional) — IDs: `1` (Accounting) to `26` (Support)
- `company_headcount` (array of strings, optional) — `"A"` (Self) to `"I"` (10K+)

**Seniority IDs:**
- `100` In Training, `110` Entry, `120` Senior, `130` Strategic
- `200` Entry Manager, `210` Experienced Manager
- `300` Director, `310` VP, `320` CXO
- `400` Owner

**Function IDs:** `1` Accounting, `2` Admin, `3` Arts, `4` Biz Dev, `5` Community, `6` Consulting, `7` Education, `8` Engineering, `9` Entrepreneurship, `10` Finance, `11` Healthcare, `12` HR, `13` IT, `14` Legal, `15` Marketing, `16` Media, `17` Military, `18` Operations, `19` Product Mgmt, `20` Program, `21` Purchasing, `22` QA, `23` Real Estate, `24` Research, `25` Sales, `26` Support

**Headcount:** `A` Self, `B` 1-10, `C` 11-50, `D` 51-200, `E` 201-500, `F` 501-1K, `G` 1K-5K, `H` 5K-10K, `I` 10K+

**Returns:** `{ success, message, job_id }`

**Example:**
```
"Search for 25 CTOs in Spain with company size 51-200"
"Find 50 VP Engineering profiles in Germany, seniority VP or CXO"
```

---

## `generate_message`

Generate or regenerate an AI message for a prospect. **Does NOT approve or send the message.** Costs credits.

**Input:**
- `prospect_id` (string, required)
- `step` (number, optional) — `0` = connection note, `1` = follow-up, `2` = value, `3` = close. Default `0`.

**Returns:** `{ prospect, step, message }`

**Example:**
```
"Generate a preview connection message for [prospect ID]"
"Create a follow-up message (step 1) for [prospect]"
```

---

## `list_recent_activity`

Get recent activity log for a company.

**Input:**
- `company_id` (string, required)
- `limit` (number, optional) — Default 20, max 50

**Returns:** Array of activity events with `action`, `details`, `source`, `created_at`, `prospect_id`.

**Example:**
```
"What happened in my account today?"
"Show me recent activity from the last 20 actions"
```

---

## What the MCP server CANNOT do

- ❌ **Approve prospects** — Manual user decision via dashboard at [app.nuph.ai](https://app.nuph.ai)
- ❌ **Send LinkedIn messages or connection requests** — Requires the official [Chrome Extension](https://chromewebstore.google.com/detail/nuphai-linkedin-outreach/ekcjniemnbdjjobajommjdnoimhdijel) installed in the browser where LinkedIn is logged in
- ❌ **Modify account settings, billing, or plan**
- ❌ **Delete prospects, campaigns, or companies**
- ❌ **Bulk operations** (intentionally — prevents AI from accidentally doing something expensive)

This is **intentional**. The MCP server is designed for:
- **Exploration** (read-heavy queries)
- **Search** (find new leads)
- **Preview** (see AI messages before committing)

Actual outreach flow stays: User reviews in dashboard → User clicks approve → Chrome extension sends from LinkedIn.

## Credit costs

Tools that cost credits:
- `search_leads` — ~2 credits per lead returned (rounded)
- `generate_message` — ~1 credit per generation

All other tools are free (read-only).

See [nuph.ai/pricing](https://nuph.ai/pricing/) for current plans.
