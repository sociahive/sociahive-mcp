# SociaHive MCP Server

Hosted remote [MCP](https://modelcontextprotocol.io) server for [SociaHive](https://www.sociahive.com) — run your social media from Claude, ChatGPT, Cursor, or any MCP-capable agent.

**Endpoint:** `https://www.sociahive.com/api/mcp` (Streamable HTTP)

70 tools covering:

- **Scheduling** — `schedule_post`, `publish_post_now`, `get_scheduler_calendar`, `create_scheduled_post`, `bulk_reschedule_posts` across 9 platforms (Instagram, Facebook, LinkedIn, X/Twitter, Threads, TikTok, YouTube, Bluesky, Pinterest)
- **DM automation** — `generate_flow` (describe an automation in plain English, get a reviewable draft), `create_automation`, `activate_flow`, `list_flows_with_stats`, node-level editing tools
- **Analytics** — `get_post_analytics`, `get_analytics_overview`, `get_flow_stats`, `get_content_insights`
- **Autopilot** — `generate_autopilot_week`, `adjust_autopilot_week`, `approve_autopilot_week` (plan and approve a full week of AI-generated content)
- **CRM & growth** — `capture_lead`, `create_broadcast`, `create_sequence`, `create_growth_widget`, `get_pixel_embed`

## Safety model

- API keys are **scoped** (`posts:read`, `posts:write`, `flows:write`, `analytics:read`, …) — a read-only key can never publish.
- Irreversible actions (publishing to a live audience, turning automations on) are **confirmation-gated**: the first call returns a short-lived confirmation token; nothing goes live until the confirming second call.
- Automations are always created as **drafts** — turning one on is an explicit act.

## Setup

Get an API key: [SociaHive → Settings → API Keys](https://www.sociahive.com/settings) (free plan works). Full per-assistant walkthroughs: [sociahive.com/connect-ai](https://www.sociahive.com/connect-ai).

### Claude Code

```bash
claude mcp add --transport http sociahive https://www.sociahive.com/api/mcp \
  --header "Authorization: Bearer sk_live_..."
```

### Claude Desktop / claude.ai

Add a custom connector with URL `https://www.sociahive.com/api/mcp`. OAuth 2.1 is supported for browser-based connect; API-key auth (`Authorization: Bearer sk_...` or `X-API-Key`) works everywhere else.

### Cursor

```json
{
  "mcpServers": {
    "sociahive": {
      "url": "https://www.sociahive.com/api/mcp",
      "headers": { "Authorization": "Bearer sk_live_..." }
    }
  }
}
```

### Agent frameworks

Works with any client that speaks Streamable HTTP — OpenAI Agents SDK (`MCPServerStreamableHttp`), LangChain (`langchain-mcp-adapters`), CrewAI. Examples: [sociahive.com/developers](https://www.sociahive.com/developers).

## CLI

The same tool surface is available as a command-line client:

```bash
export SOCIAHIVE_API_KEY=sk_live_...
sociahive posts schedule <id> --at "2026-09-01T15:00:00Z"
sociahive flows generate --description "DM my price list when someone comments PRICE"
sociahive insights --weeks 8
```

## Links

- Product: [sociahive.com](https://www.sociahive.com)
- MCP overview: [sociahive.com/mcp](https://www.sociahive.com/mcp)
- Developer docs: [sociahive.com/developers](https://www.sociahive.com/developers) · [docs.sociahive.com](https://docs.sociahive.com)
- Connect walkthroughs per assistant: [sociahive.com/connect-ai](https://www.sociahive.com/connect-ai)

The server itself is part of the SociaHive platform (closed source); this repository hosts the public manifest (`server.json`), setup documentation, and client configuration examples.
