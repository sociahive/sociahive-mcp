---
name: sociahive
description: Use SociaHive to schedule and publish social media posts to Instagram, TikTok, X (Twitter), LinkedIn, Facebook, Threads, YouTube, Bluesky and Pinterest; to build and run DM and comment automations (comment-to-DM, keyword replies, lead capture) on Instagram, Facebook, Messenger, WhatsApp and Telegram; and to read post, automation and account analytics. Use it when the user asks to post, schedule, plan a content calendar, upload media for a post, set up an automation that replies to comments or DMs, collect leads, or check how their posts and automations perform.
homepage: https://www.sociahive.com/connect-ai
allowed-tools: Bash(sociahive:*)
---

# SociaHive

SociaHive is a social media platform for creators and small teams: one place to schedule posts on 9 networks, automate DMs and comment replies, and see what works.

There are two ways to drive it. Prefer the MCP tools when they are connected; use the `sociahive` CLI for local files and scripted bulk work.

## 1. MCP server (preferred)

- Endpoint: `https://www.sociahive.com/api/mcp` (streamable HTTP).
- Sign-in: OAuth 2.1 with dynamic client registration. The first call returns 401 and the client opens a SociaHive sign-in page. No token goes in any config file.
- If SociaHive tools are listed (`list_accounts`, `schedule_post`, `generate_flow`, ...), use them directly.
- If they are not listed, ask the user to connect SociaHive in their agent (see https://www.sociahive.com/connect-ai) or fall back to the CLI.

## 2. CLI

```bash
npm install -g sociahive          # or run without installing: npx sociahive <command>
export SOCIAHIVE_API_KEY=sk_...   # https://www.sociahive.com/settings?tab=api-keys
sociahive --help
```

The key needs the `agent:execute` scope. Never print the key or write it into files the user did not ask for.

Use the CLI when:

- A post needs local images or videos: `sociahive upload <file> [<file> ...]` prints the hosted URLs, or pass `--media-file <path>` to `posts create`. (Over MCP, `upload_media_from_url` copies a file from a public URL.)
- The user has a CSV of posts: `sociahive posts import <file.csv>` (columns `text`, `platforms` separated by `|`, `media` URLs separated by `|`). Every row becomes a draft.
- The work is a script or a loop over many posts or flows.

## Plan requirement

MCP, API key and CLI access are part of the 7-day free trial and every paid plan. On the free plan every tool call answers with an upgrade message and a billing link; relay it to the user and stop. Do not retry.

## Safety rules

- Publishing to a live audience and other irreversible actions need the user's explicit "yes". Over MCP these tools return a confirmation request first: show the user what will happen, and only after they agree call again with the confirmation token. In the CLI, only add `--yes` after the user has agreed.
- Automations are created as drafts. Turning one on (`activate_flow`) makes it reply to real people; confirm with the user first.
- Prefer scheduling over publishing now unless the user asks for now.
- Check `list_accounts` before posting: use only accounts that are connected, and pass their ids.
- Respect each platform's limits (X 280 characters and up to 4 images, Instagram captions 2,200, LinkedIn 3,000). The server refuses text that does not fit and says which target failed.

## MCP tools by job

### Account and setup
- `whoami`: who is signed in.
- `list_accounts`: connected social accounts and their ids.
- `get_capabilities`, `list_capabilities`: what each account and plan can do.

### Posts and scheduling
- `create_scheduled_post`: create a post (draft or scheduled) for one or more accounts, with per-platform text and media.
- `schedule_post`, `update_scheduled_post`, `cancel_scheduled_post`: schedule a draft, edit it, cancel it.
- `publish_post_now`, `publish`: publish immediately. Irreversible, asks for confirmation.
- `list_scheduled_posts`, `get_scheduled_post`, `get_scheduler_calendar`: see what is planned.
- `bulk_reschedule_posts`: move many posts at once.
- `upload_media_from_url`: copy an image or video from a public URL into SociaHive for a post.
- `add_to_evergreen`, `list_evergreen`: keep posts in a queue that is reused when the calendar has gaps.

### Published content
- `list_published_media`, `get_latest_published_media`, `get_published_media`, `search_published_media`: posts already live on the user's accounts (for example to attach an automation to a reel).

### Automations (DM and comment flows)
- `generate_flow`: describe an automation in plain words ("DM my price list when someone comments PRICE") and get a reviewable draft.
- `generate_multiple_flows`: rebuild a whole setup (for example from another tool) as several drafts. Asks for confirmation.
- `create_automation`, `create_flow`: create a flow directly.
- `list_flows`, `list_flows_with_stats`, `get_flow`, `search_flows`, `summarize_flow_purpose`: find and explain flows.
- `update_flow`, `duplicate_flow`, `bulk_update_flows`: edit flows.
- `add_node`, `update_node`, `delete_node`, `add_edge`, `delete_edge`: edit the steps of a flow.
- `activate_flow`, `deactivate_flow`: turn a flow on or off.
- `get_flow_revisions`, `revert_flow_to_version`: history and undo.
- `list_system_templates`: ready-made automation templates.

### Leads, contacts and growth
- `get_collected_data`: what a flow has collected (field types and counts).
- `capture_lead`: record a lead for a contact.
- `resolve`: answer a support conversation from the knowledge base when confident.
- `create_broadcast`, `create_sequence`: one-off and drip messages to contacts.
- `create_landing_page`, `create_growth_widget`, `create_ref_url`, `get_pixel_embed`: entry points that bring people into automations.
- `search_tags`, `create_tag`, `find_references`: contact tags and where they are used.

### Analytics
- `get_post_analytics`, `get_analytics_overview`: post and account performance.
- `get_flow_stats`: runs and results of an automation.
- `get_content_insights`: what kind of content works best for this account.

## CLI commands

```bash
sociahive whoami
sociahive accounts list
sociahive accounts capabilities [--account-id <id>]

sociahive upload <file> [<file> ...]
sociahive posts create (--text "..." | --text-file <path>) --platforms '[{"platform":"instagram","accountId":"..."}]' \
  [--scheduled-at "2026-11-01T15:00:00Z"] [--timezone UTC] [--media-file <path> ...]
sociahive posts list [--status <s>] [--platform <p>] [--limit <n>]
sociahive posts get <id>
sociahive posts update <id> [--text "..."] [--scheduled-at "..."]
sociahive posts schedule <id> --at "2026-11-01T15:00:00Z" [--timezone UTC]
sociahive posts publish <id> --yes          # irreversible, only after the user agrees
sociahive posts cancel <id>
sociahive posts import <file.csv>
sociahive calendar --month 2026-11 [--timezone UTC]
sociahive evergreen list
sociahive evergreen add --text "Caption to recycle"

sociahive flows generate --description "DM my price list when someone comments PRICE" [--platform instagram]
sociahive flows bulk-generate --description "..." --yes
sociahive flows list [--status draft|published|all]
sociahive flows get <id>
sociahive flows activate <id>
sociahive flows deactivate <id>
sociahive flows duplicate <id>
sociahive flows stats <id>
sociahive templates list

sociahive analytics [--post-id <id>] [--start-date YYYY-MM-DD] [--end-date YYYY-MM-DD]
sociahive insights [--weeks 8]
```

Per-platform text and media go in each `--platforms` target (`"text"`, `"media"`, `"contentType"`, `"metadata"`). Run `sociahive <command> --help` for every option.

## Links

- Connect guides per agent: https://www.sociahive.com/connect-ai
- Developer docs: https://www.sociahive.com/developers
- API keys: https://www.sociahive.com/settings?tab=api-keys
