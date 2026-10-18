# SociaHive for AI agents

Hosted remote [MCP](https://modelcontextprotocol.io) server, skill and plugins for [SociaHive](https://www.sociahive.com): schedule and publish posts to Instagram, TikTok, X, LinkedIn, Facebook, Threads, YouTube, Bluesky and Pinterest, build DM and comment automations, and read analytics from Claude Code, Codex, Gemini CLI, Cursor, Grok, Claude, ChatGPT or any MCP client.

**Endpoint:** `https://www.sociahive.com/api/mcp` (streamable HTTP). Sign in with OAuth 2.1 on first use, or send an API key (`X-API-Key: sk_...` or `Authorization: Bearer sk_...`).

MCP, API key and CLI access are part of the 7-day trial and every paid plan.

This repository holds:

- `server.json`: the MCP registry manifest.
- `skills/sociahive/SKILL.md`: which tool to use for which job, the CLI commands and the safety rules (copied to the root `SKILL.md`).
- Plugin manifests for Claude Code (`.claude-plugin/`), Cursor (`.cursor-plugin/`), Codex (`.codex-plugin/`, `.agents/plugins/`), Grok Build (`.grok-plugin/`) and a Gemini CLI extension (`gemini-extension.json`, `GEMINI.md`).

## What it can do

- **Scheduling:** `create_scheduled_post`, `schedule_post`, `publish_post_now`, `get_scheduler_calendar`, `bulk_reschedule_posts`, `upload_media_from_url`
- **DM and comment automation:** `generate_flow` (describe an automation in plain words, get a reviewable draft), `generate_multiple_flows`, `create_automation`, `activate_flow`, `list_flows_with_stats`, step-level editing
- **Analytics:** `get_post_analytics`, `get_analytics_overview`, `get_flow_stats`, `get_content_insights`
- **Leads and growth:** `capture_lead`, `get_collected_data`, `create_broadcast`, `create_sequence`, `create_growth_widget`, `get_pixel_embed`

## Install

### Claude Code

```bash
/plugin marketplace add sociahive/sociahive-mcp
/plugin install sociahive@sociahive-mcp
```

Or only the MCP server:

```bash
claude mcp add --transport http sociahive https://www.sociahive.com/api/mcp
```

### Codex

```bash
codex plugin marketplace add sociahive/sociahive-mcp
```

Or only the MCP server, then sign in:

```bash
codex mcp add sociahive --url https://www.sociahive.com/api/mcp
codex mcp login sociahive
```

### Gemini CLI

```bash
gemini extensions install https://github.com/sociahive/sociahive-mcp
```

Or only the MCP server:

```bash
gemini mcp add --transport http sociahive https://www.sociahive.com/api/mcp
```

### Cursor

[Add to Cursor](cursor://anysphere.cursor-deeplink/mcp/install?name=sociahive&config=eyJ1cmwiOiJodHRwczovL3d3dy5zb2NpYWhpdmUuY29tL2FwaS9tY3AifQ==), or add this to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "sociahive": { "url": "https://www.sociahive.com/api/mcp" }
  }
}
```

### Claude and ChatGPT

Add a custom connector with the URL `https://www.sociahive.com/api/mcp` and sign in when asked.

### Any agent that reads skills

```bash
npx skills add sociahive/sociahive-mcp
```

### Any MCP client, one command

```bash
npx @sociahive/mcp add
```

The wizard detects your agents and writes their MCP config.

### Agent frameworks

Works with any client that speaks streamable HTTP: OpenAI Agents SDK (`MCPServerStreamableHttp`), LangChain (`langchain-mcp-adapters`), CrewAI. Examples: [sociahive.com/developers](https://www.sociahive.com/developers).

## The CLI

For local images and videos, CSV imports and scripts:

```bash
npm install -g sociahive
export SOCIAHIVE_API_KEY=sk_...   # https://www.sociahive.com/settings?tab=api-keys
sociahive upload slide_01.jpg slide_02.jpg
sociahive flows generate --description "DM my price list when someone comments PRICE"
sociahive insights --weeks 8
```

## Safety

- API keys are scoped (`posts:read`, `posts:write`, `flows:write`, `analytics:read`, ...); a read-only key can never publish. OAuth sign-in offers read-only access.
- Publishing to a live audience and turning an automation on ask for confirmation first: nothing goes live until a second, confirming call.
- Automations are created as drafts for you to review.

## Links

- Connect guides: [sociahive.com/connect-ai](https://www.sociahive.com/connect-ai)
- Developer docs: [sociahive.com/developers](https://www.sociahive.com/developers) · [docs.sociahive.com](https://docs.sociahive.com)
- [Privacy](https://www.sociahive.com/privacy) · [Terms](https://www.sociahive.com/terms)

The server is part of the SociaHive platform (closed source). This repository holds the public manifests, the skill and setup docs. MIT licensed.
