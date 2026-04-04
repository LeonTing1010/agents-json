# agents.json

**A web standard for AI agent-friendly websites.**

`robots.txt` tells agents what they *cannot* do.
`agents.json` tells agents what they *can* do — and how to do it well.

## The Problem

AI agents automate websites by inspecting pages, guessing API endpoints, and reverse-engineering data structures. This is fragile, slow, and adversarial.

## The Solution

Websites publish `/.well-known/agents.json` — a machine-readable file declaring their automation-friendly interfaces. Agents consume it and skip the guesswork.

```
GET https://example.com/.well-known/agents.json
```

## Minimal Example (3 fields)

```json
{
  "version": "1.0",
  "name": "Hacker News",
  "description": "Tech news aggregator with community voting"
}
```

This is a valid `agents.json`. Start here, add more as needed.

## Full Example

```json
{
  "version": "1.0",
  "name": "Hacker News",
  "description": "Tech news aggregator with community voting",
  "homepage": "https://news.ycombinator.com",

  "data": [
    {
      "name": "top-stories",
      "description": "Front page stories ranked by score",
      "endpoint": "https://hacker-news.firebaseio.com/v0/topstories.json",
      "method": "GET",
      "format": "json",
      "fields": {
        "id": "number — story ID",
        "title": "string — story title",
        "url": "string — link URL",
        "score": "number — upvote count",
        "by": "string — author username"
      },
      "auth": "none",
      "rate_limit": "30 req/min"
    }
  ],

  "actions": [
    {
      "name": "submit-story",
      "description": "Submit a new story",
      "endpoint": "https://news.ycombinator.com/submit",
      "method": "POST",
      "auth": "cookie",
      "fields": {
        "title": "string — story title (required)",
        "url": "string — link URL (optional)"
      },
      "requires_confirmation": true
    }
  ],

  "specs": {
    "openapi": "https://api.example.com/openapi.json",
    "llms_txt": "https://example.com/llms.txt",
    "rss": "https://example.com/rss"
  },

  "policies": {
    "automation_allowed": true,
    "rate_limit": "60 req/min",
    "requires_attribution": false,
    "contact": "api@example.com"
  }
}
```

## Who Benefits

| Stakeholder | Benefit |
|-------------|---------|
| **Websites** | Control how agents interact. Set rate limits, require attribution, block bad actors. |
| **AI Agents** | Skip page inspection. Go straight to the right endpoint with correct field names. |
| **Users** | Faster, more reliable automations that don't break when sites redesign. |

## Field Reference

### Top-level (required)

| Field | Type | Description |
|-------|------|-------------|
| `version` | string | Spec version (`"1.0"`) |
| `name` | string | Site name |
| `description` | string | One-line description |

### `data[]` — Read Endpoints

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Identifier (e.g. `"top-stories"`) |
| `description` | string | yes | What this data represents |
| `endpoint` | string | yes | Full URL or relative path |
| `method` | string | no | HTTP method (default: `GET`) |
| `format` | string | no | `json`, `html`, `xml`, `rss`, `csv` |
| `fields` | object | no | Field name to `"type — description"` |
| `auth` | string | no | `none`, `cookie`, `bearer`, `api_key` |
| `rate_limit` | string | no | Human-readable rate limit |

### `actions[]` — Write Operations

Same as `data[]` plus:

| Field | Type | Description |
|-------|------|-------------|
| `requires_confirmation` | boolean | Agent should confirm with user first |

### `specs` — Pointers to Detailed Specs

| Field | Type | Description |
|-------|------|-------------|
| `openapi` | string | URL to OpenAPI spec |
| `graphql` | string | URL to GraphQL endpoint |
| `llms_txt` | string | URL to llms.txt |
| `rss` | string | URL to RSS/Atom feed |

### `policies` — Automation Rules

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `automation_allowed` | boolean | `true` | AI automation permitted |
| `rate_limit` | string | — | Global rate limit |
| `requires_attribution` | boolean | `false` | Must credit source |
| `allowed_agents` | string[] | `[]` | Whitelist (empty = all) |
| `blocked_agents` | string[] | `[]` | Blacklist |
| `contact` | string | — | Email for inquiries |

## Relationship to Other Standards

| Standard | What it does | How agents.json relates |
|----------|-------------|------------------------|
| `robots.txt` | Deny list for crawlers | agents.json is the allow list |
| `llms.txt` | LLM-readable site description | `specs.llms_txt` points to it |
| OpenAPI | Full API specification | `specs.openapi` points to it |
| A2A `agent-card.json` | Agent declares its capabilities | Complementary — agents.json is the website's side |
| Schema.org | Semantic page data | agents.json describes endpoints; Schema.org describes content |

## Adopters

| Site | agents.json |
|------|-------------|
| [taprun.dev](https://taprun.dev) | [/.well-known/agents.json](https://taprun.dev/.well-known/agents.json) |

## How to Adopt

1. Create `/.well-known/agents.json` on your site
2. Start with `version`, `name`, `description` (3 fields)
3. Add `data` entries for your public endpoints
4. Add `policies` to set rate limits
5. Submit a PR to add your site to the Adopters list

## Status

**Draft** — Community feedback welcome. [Open an issue](https://github.com/LeonTing1010/agents-json/issues) or submit a PR.

Future: IANA well-known URI registration (RFC 8615) once sufficient adoption.

## License

[CC0 1.0 Universal](LICENSE) — No rights reserved. Use freely.
