# Protocol Reference

Low-level JSON-RPC 2.0 protocol details for the nuph.ai MCP Server.

Most users don't need this — your MCP client handles the protocol automatically. This doc is for developers building custom integrations.

## Overview

| | |
|---|---|
| **Server version** | v2.1.0 |
| **Protocol** | [Model Context Protocol](https://modelcontextprotocol.io/) v2025-06-18 |
| **Transport** | Streamable HTTP |
| **Format** | [JSON-RPC 2.0](https://www.jsonrpc.org/specification) |
| **Endpoint** | `POST https://api.nuph.ai/functions/v1/outreach-mcp` |
| **Content-Type** | `application/json` |
| **Auth** | `Authorization: Bearer <api_key>` (or `?apikey=...` query param) |

## Request format

Every request is a JSON-RPC 2.0 message:

```json
{
  "jsonrpc": "2.0",
  "method": "<method_name>",
  "params": { /* method-specific */ },
  "id": 1
}
```

## Response format

Success:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": { /* method-specific */ }
}
```

Error:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32000,
    "message": "Error description"
  }
}
```

## Supported methods

### `initialize`

Initialize the MCP session. Client sends this first.

**Request:**
```json
{
  "jsonrpc": "2.0",
  "method": "initialize",
  "id": 1
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "protocolVersion": "2025-06-18",
    "capabilities": {
      "tools": {}
    },
    "serverInfo": {
      "name": "nuph.ai",
      "version": "2.1.0"
    }
  }
}
```

### `notifications/initialized`

Client notifies that initialization is complete. No response expected (it's a notification).

### `tools/list`

Get the list of available tools.

**Request:**
```json
{
  "jsonrpc": "2.0",
  "method": "tools/list",
  "id": 2
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "tools": [
      {
        "name": "list_companies",
        "description": "List all companies owned by the authenticated user",
        "inputSchema": { "type": "object", "properties": {}, "required": [] }
      },
      // ... 46 more tools
    ]
  }
}
```

### `tools/call`

Execute a tool.

**Request:**
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "list_prospects",
    "arguments": {
      "company_id": "abc-123",
      "min_score": 8,
      "limit": 10
    }
  },
  "id": 3
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "[{\"id\":\"p1\",\"first_name\":\"Anna\",...},...]"
      }
    ]
  }
}
```

The `content` field is an array of content blocks. Currently all responses return a single text block containing JSON-stringified data.

### `ping`

Health check.

**Request:**
```json
{
  "jsonrpc": "2.0",
  "method": "ping",
  "id": 4
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 4,
  "result": {}
}
```

## Error codes

Standard JSON-RPC error codes:

| Code | Meaning | When it happens |
|------|---------|----------------|
| `-32700` | Parse error | Invalid JSON |
| `-32600` | Invalid request | Missing `jsonrpc: "2.0"` field |
| `-32601` | Method not found | Unknown method or tool name |
| `-32602` | Invalid params | Missing tool name in `tools/call` |
| `-32000` | Server error | Auth failure, missing API key |

Tool-specific errors return `result` with `isError: true`:

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "result": {
    "content": [
      { "type": "text", "text": "Error: prospect not found or access denied" }
    ],
    "isError": true
  }
}
```

## Authentication

Include your API key as a Bearer token:

```
Authorization: Bearer outreach_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

The server also accepts `x-api-key` header for backwards compatibility:

```
x-api-key: outreach_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

## CORS

The server allows cross-origin requests from:
- `https://nuph.ai`
- `https://app.nuph.ai`
- `http://localhost:*` (for development)

Other origins are rejected. MCP clients don't need to worry about this (they're not browsers).

## Rate limiting

Per-API-key rate limit (independent of credit balance):
- 60 calls/minute
- 600 calls/hour

Configurable per-account via `outreach_ai_config.mcp_max_calls_per_min` and `mcp_max_calls_per_hour`. When exceeded the server returns HTTP 429 with a `Retry-After` header AND a JSON-RPC error:

```json
{
  "jsonrpc": "2.0",
  "id": null,
  "error": {
    "code": -32029,
    "message": "Rate limit: 60 calls/minute exceeded. Retry in 60 seconds.",
    "data": {
      "code": "rate_limited",
      "retry_after_seconds": 60
    }
  }
}
```

Tools that cost credits (`search_leads`, `enrich_prospects`, `generate_message`, etc.) deduct on use. Read-only tools have no credit cost.

## dry_run contract for write tools

Every write tool defaults to `dry_run: true`. The first call returns a structured preview instead of executing:

```json
{
  "jsonrpc": "2.0",
  "id": 4,
  "result": {
    "content": [{
      "type": "text",
      "text": "{ \"preview\": {...}, \"missing_required\": [], \"missing_recommended\": [...], \"warnings\": [], \"estimated_credits\": null, \"preflight\": {...}, \"ready_to_execute\": true, \"next_call\": { \"tool\": \"create_campaign\", \"arguments\": {...}, \"with\": { \"dry_run\": false } } }"
    }]
  }
}
```

Pass `dry_run: false` in the next call to actually write. The model can build the second call from the `next_call` template.

Override globally via env: set `MCP_DRY_RUN_DEFAULT=false` on the function. Not recommended (defeats the safety pattern).

## Structured errors

Tool-specific errors return `result` with `isError: true` AND a `structured_error` object with stable error codes the model can branch on:

```json
{
  "isError": true,
  "content": [{ "type": "text", "text": "User owns multiple companies. Ask which one to use." }],
  "structured_error": {
    "code": "ambiguous_company",
    "help": "User owns multiple companies. Ask which one to use, then call again with company_id.",
    "details": {
      "options": [
        { "id": "abc-123", "name": "Acme" },
        { "id": "def-456", "name": "Globex" }
      ]
    },
    "retry_with": { "field": "company_id", "from": "options[].id" }
  }
}
```

Stable error codes: `unauthorized`, `not_found`, `no_company`, `ambiguous_company`, `ambiguous_campaign`, `missing_required`, `validation_failed`, `insufficient_credits`, `rate_limited`, `dry_run_required`, `state_changed`, `transitional_status_blocked`, `internal_error`.

## Timeouts

- Request timeout: 60 seconds
- Idle connection timeout: 120 seconds

## Examples with cURL

### Initialize

```bash
curl -X POST https://api.nuph.ai/functions/v1/outreach-mcp \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"initialize","id":1}'
```

### List tools

```bash
curl -X POST https://api.nuph.ai/functions/v1/outreach-mcp \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/list","id":2}'
```

### Call a tool

```bash
curl -X POST https://api.nuph.ai/functions/v1/outreach-mcp \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"get_credits_balance","arguments":{}},"id":3}'
```

## Discovery

The server supports MCP discovery via a well-known URL:

```
GET https://nuph.ai/.well-known/mcp.json
```

Returns a server card with transport, capabilities, auth type, and tool list.
