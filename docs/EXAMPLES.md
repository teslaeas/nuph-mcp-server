# Example Prompts

Tested prompts for the nuph.ai MCP Server v2.1.0 (47 tools), organized by use case.

## Best first call (always)

```
"Give me an overview of my nuph.ai workspace"
   → get_workspace_overview()
   → returns companies, active campaigns, credits, recent jobs, alerts in 1 call

"What should I do today?"
   → suggest_next_actions()
   → returns prioritized list with pre-computed tool calls
```

## Getting started

```
"What nuph.ai tools do you have?"
"What write actions does the MCP support?"   → list_form_specs()
"List my companies"
"How many credits do I have left?"
"What plan am I on?"
```

## Exploring your pipeline

```
"Show me my active campaigns"
"Which campaign has the most prospects?"
"What's the ICP profile for my top campaign?"
"List all my campaigns and their stats"
```

## Finding top prospects

All these use `query_prospects` (the unified smart search). Returns `linkedin_url` for every prospect.

```
"Show me my top 10 prospects with score above 8"
"Show me ALL my connected prospects"           ← returns the real total, no max-50 cap
"List all prospects with score 9 or 10"
"Find my connected prospects with score above 7"
"Show me prospects in the 'Germany CTO' campaign with min_score 8"
"Who are my newest prospects this week?"
"Find prospects who replied in the last 14 days"
"Get prospects with verified emails in Spain"
```

## Smart pitch matching (no AI cost)

Pure SQL ranking against a free-text pitch. Returns `match_score` + `match_reasoning`.

```
"Find prospects best fit for AI outreach automation for SaaS sales"
   → find_pitchable_prospects(pitch: "AI outreach automation for SaaS sales")

"Who can I pitch our marketing analytics tool to?"
   → find_pitchable_prospects(pitch: "marketing analytics tool")

"Search prospects with title containing 'CTO' or 'VP' in Spain or Germany"
   → query_prospects(filters: { title_contains: ["CTO","VP"], country: ["Spain","Germany"] })
```

## Searching LinkedIn

Basic:
```
"Search for 25 CTOs in Spain"
"Find 50 VP Engineering profiles in Germany"
"Search for Head of Sales in the United States"
```

With filters:
```
"Search for 25 CTOs in Germany with company size 51-200"
"Find VPs of Engineering in Berlin, seniority CXO or VP"
"Search for 25 Sales Directors in the US, Engineering or IT function"
"Find Marketing Managers in Spain, company size 11-50, small companies only"
```

Multi-location:
```
"Search for 25 CTOs across Spain, Germany, and France"
"Find Head of Sales in US and UK"
```

## Getting prospect details

```
"Tell me everything about [prospect ID]"
"What messages have been generated for [prospect name]?"
"Show me the profile of my highest scoring prospect"
"Who is [prospect name] and what's their background?"
```

## Generating message previews

Remember: `generate_message` only creates the message — you must approve from the dashboard to actually send.

```
"Generate a preview connection message for [prospect ID]"
"Create a follow-up message (step 1) for [prospect name]"
"Show me what a value message (step 2) would look like for this prospect"
"Generate a close message for my top prospect"
```

## Analysis and insights

```
"Which of my campaigns has the highest approval rate?"
"How many prospects did I get this month?"
"What's my overall conversion rate from new to connected?"
"Show me recent activity from the last 20 actions"
"What happened in my account today?"
"How have my credits been used this week?"
```

## Multi-step workflows

Claude can chain multiple tools:

```
"Find me 25 CTOs in Spain and then show me which ones have the highest scores"

→ Claude runs: search_leads → (wait) → list_prospects with min_score

"Look at my [campaign name] campaign and tell me which prospects are worth reaching out to first"

→ Claude runs: get_campaign → list_prospects filtered by that campaign

"Analyze my pipeline: how many prospects do I have per company, and what's the average score?"

→ Claude runs: list_companies → list_prospects (per company) → aggregates
```

## Real use cases

### Daily standup
```
"What happened in my nuph.ai account yesterday? List new prospects added, messages generated, and searches launched."
```

### Campaign review
```
"Give me a full breakdown of my [campaign name] campaign: total prospects, scores distribution, top 5 leads, and their current status."
```

### Lead prioritization
```
"I have 30 minutes to do outreach. Based on score and my credit balance, which 5 prospects should I focus on approving in the dashboard?"
```

### Comparative analysis
```
"Compare my top 3 campaigns. Which has the best ICP fit, most engaged prospects, and highest conversion rate?"
```

### Budget planning
```
"I have 500 credits. Given my typical search costs, how many leads can I realistically find this month? Break it down by campaign."
```

## Tips for better prompts

- **Be specific about filters** — "CTOs in Spain" is better than "tech leads somewhere"
- **Mention score thresholds** — "score > 7" filters noise
- **Use prospect IDs for precision** — Ask "show me [ID]" rather than fuzzy matching names
- **Let Claude chain tools** — "Find and show me" is better than running each manually
- **Set a goal** — "I want to find 10 high-quality CTOs to approach this week" gives Claude context to help strategically

## Job monitoring & cancellation

```
"What scraping jobs are running right now?"
   → list_recent_jobs(status: "running")
   → response includes a tip: "Active jobs in flight. Poll list_recent_jobs every 30-60s until completed."

"Show me my last 10 search jobs"
   → list_recent_jobs(limit: 10)

"Cancel job [ID] — it's still pending and I want my credits back"
   → cancel_job(job_id: "...")        ← preview first
   → cancel_job(job_id: "...", dry_run: false)
```

## Conversational creates (preview-then-confirm)

Every write tool returns a preview first. The model surfaces missing fields and asks before executing.

```
"Create a campaign called Q3 Spain"
   → create_campaign(name: "Q3 Spain")
   → returns preview with missing_recommended: [icp_profile_id, language, tone_id, target_market]
   → Claude asks: "Which ICP, language, and tone? Markets to target?"

[user answers...]

"Use Pro tone, link Marketing Leads EU, in English, target Spain only"
   → create_campaign(..., dry_run: false)
   → executes; returns { result, credits_consumed: 0 }
```

## Multi-company users (auto-resolve)

```
"List my campaigns"   ← if you own >1 company:
   → list_campaigns()
   → structured error ambiguous_company with full list of {id, name}
   → Claude asks: "Which company? You have: Acme, Globex, Initech."

"Use Acme"
   → list_campaigns(company_id: "<acme>")
   → returns campaigns with resolved_context: { company_id, auto_resolved: false }
```

## What to avoid

- ❌ "Approve prospect X" — Not supported (do it in the dashboard)
- ❌ "Send a message to X" — Not supported (Chrome extension required)
- ❌ "Buy 500 credits" — Not supported (billing is dashboard-only)
- ❌ "Modify my plan / billing" — Not supported

> **Note:** `delete_campaign` and `delete_company` ARE supported (with `confirm: true` and `dry_run: false`), but use with care — they cascade-delete prospects and messages.

## Contributing examples

Have a useful prompt? [Open a PR](https://github.com/teslaeas/nuph-mcp-server/pulls) to add it here!
