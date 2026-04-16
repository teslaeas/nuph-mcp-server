# Example Prompts

Tested prompts for the nuph.ai MCP Server, organized by use case.

## Getting started

```
"What nuph.ai tools do you have?"
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

```
"Show me my top 10 prospects with score above 8"
"List all prospects with score 9 or 10"
"Find my connected prospects with score above 7"
"Show me prospects in the 'Germany CTO' campaign with min_score 8"
"Who are my newest prospects this week?"
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

## What to avoid

- ❌ "Approve prospect X" — Not supported (do it in the dashboard)
- ❌ "Send a message to X" — Not supported (Chrome extension required)
- ❌ "Delete campaign X" — Not supported
- ❌ "Buy 500 credits" — Not supported (billing is dashboard-only)

## Contributing examples

Have a useful prompt? [Open a PR](https://github.com/nuph-ai/nuph-mcp-server/pulls) to add it here!
