# SociaHive tool catalog

Tools a normal paid or trial account can call over MCP. `scope` is the API key scope; one-click Connect keys and OAuth sign-in carry `agent:execute`, `accounts:read`, `posts:read`, `posts:write`, `flows:read`, `flows:write`, `analytics:read`, `templates:read`, `uploads:write`. Tools needing another scope work only with a custom API key (Settings, API keys).

Ids: `postId` is a scheduler post id; `flowId` a flow id; `accountId` the `id` from `list_accounts`. Where a tool accepts `postName` or `flowName` (the caption's opening words, the flow's name), use it instead of looking an id up. Rate limits: read 120/min, write 30/min, expensive 5/min, 1,000/day.

## Session

| Tool | Purpose | Key args | Scope |
|---|---|---|---|
| `whoami` | Signed-in user, tier, plan limits. Call first. | none | none |
| `list_capabilities` | Tools this key can call. | none | none |

## Accounts and media

| Tool | Purpose | Key args | Scope |
|---|---|---|---|
| `list_accounts` | Connected accounts with `id`, `platform`, `username`, `status` (`connected`, `needs_reauth`, ...). Use only when asked what is connected, or to resolve `ambiguous_reference`. | `platform`, `status` | accounts:read |
| `get_capabilities` | What each account can do (automation vs scheduling). | `accountId` | accounts:read |
| `list_published_media` | Recent posts and reels live on the platform (Instagram first). | `mediaType`, `limit`, `accountId` | accounts:read |
| `get_latest_published_media` | The single newest live post or reel ("my latest reel"). | `mediaType`, `accountId` | accounts:read |
| `get_published_media` | One live item by the platform's media id. | `mediaId`, `accountId` | accounts:read |
| `search_published_media` | Semantic search over live captions. Counts as expensive. | `query`, `mediaType`, `limit`, `accountId` | accounts:read |
| `upload_media_from_url` | Copy a public JPEG, PNG, WebP, MP4 or MOV into SociaHive; returns `url`, `mediaType`. 30 per hour. | `url` | uploads:write |

The four published-media tools need `accountId` when the user has several accounts.

## Posts and scheduling

| Tool | Purpose | Key args | Scope | Confirm |
|---|---|---|---|---|
| `create_scheduled_post` | New post. No `scheduledAt` = draft, with it = scheduled. | `text`, `platforms[]` (`platform`, optional `accountId`, `contentType` `carousel`, per-target `text`, `media`), `media[]` (`type`, `url`, `altText`), `scheduledAt`, `timezone` | posts:write | no |
| `schedule_post` | Schedule an existing draft; can attach a comment automation (left off). | `postId` or `postName`, `scheduledAt`, `timezone`, `attach_automation` (one of `keyword`+`link`, `prompt`, `template_id`+`inputs`; optional `accountId`) | posts:write (+ flows:write when attaching) | no |
| `update_scheduled_post` | Edit or reschedule a draft or scheduled post. Media replaces all. | `postId` or `postName`, `text`, `media`, `scheduledAt`, `timezone` | posts:write | no |
| `cancel_scheduled_post` | Scheduled to draft. Idempotent. | `postId` or `postName` | posts:write | no |
| `bulk_reschedule_posts` | Move up to 50 posts; per-post failures are listed. | `postIds[]`, `scheduledAt`, `timezone` | posts:write | no |
| `publish_post_now` | Publish immediately. Irreversible. | `postId` or `postName` | posts:write | yes |
| `publish` | Same outcome as `publish_post_now`; prefer that one. | `postId`, `platforms` | posts:write | yes |
| `list_scheduled_posts` | Drafts, scheduled, published, failed. | `status` (`draft`, `scheduled`, `published`, `failed`, `all`), `platform`, `limit`, `sortOrder`, `cursor` | posts:read | no |
| `get_scheduled_post` | One post with per-target status and its automation. | `postId` | posts:read | no |
| `get_scheduler_calendar` | Posts in a date range, 200 max. | `fromDate`, `toDate` (ISO), `platform` | posts:read | no |
| `add_to_evergreen`, `list_evergreen` | Queue of recyclable captions that fills gaps in the calendar. | `text`, `sourcePostId` | posts:write / posts:read | no |

`scheduledAt` accepts ISO 8601 or `tomorrow 9am`, `friday 5pm`, `next monday`, `in 2 hours`, resolved in `timezone` (UTC if omitted).

## Analytics

| Tool | Purpose | Key args | Scope |
|---|---|---|---|
| `get_analytics_overview` | Totals, per-platform counts and a 30 day timeline. | `startDate`, `endDate` (YYYY-MM-DD), `platform` | posts:read |
| `get_post_analytics` | Impressions, reach, engagement, likes, comments, shares, saves, clicks for one published post. | `postId` | posts:read |
| `get_content_insights` | Which formats, weekdays and pillars beat the account average, with sample sizes, plus top and bottom posts. | `weeks` (1 to 12) | analytics:read |
| `get_flow_stats` | Triggers, button clicks, unique users, status breakdown, 7 day timeline for one flow. | `flowId` | flows:read |
| `list_flows_with_stats` | Rank flows by a metric. | `metric` (`triggers`, `completions`, `unique_users`, `button_clicks`), `sortOrder`, `limit`, `status` | flows:read |
| `get_collected_data` | What a flow collected: counts and field types only, never the values. | `flowId`, `fieldType`, `limit` | flows:read |

## Automations (flows)

Flow platforms: `instagram`, `whatsapp`, `messenger`, `telegram`. A flow belongs to one connected account.

| Tool | Purpose | Key args | Scope | Confirm |
|---|---|---|---|---|
| `generate_flow` | Build a draft from a description, in the background (10 to 30 s). Expensive bucket. | `description` (10 to 2,000 chars), `platform`, `accountId`, `businessContext`, `variants` (1 to 3), `iterateOnFlowId` (rebuilds a flow the AI made and nobody edited) | flows:write | no |
| `create_automation` | Draft from an outcome sentence, returns a plain-language preview. Never goes live. | `outcome`, `accountId` | flows:write | no |
| `generate_multiple_flows` | Rebuild a whole setup (up to 10 drafts), for example migrating from another tool. Starter plan and up. | `description`, `accountId` | flows:write | yes |
| `create_flow` | Empty draft to build node by node. | `name`, `accountId`, `platformUserId`, `platform`, `trigger` | flows:write | no |
| `list_flows` | Browse flows. | `status`, `limit`, `sortBy`, `sortOrder`, `cursor` | flows:read | no |
| `search_flows` | Find a flow by name. | `query`, `limit`, `accountId` | flows:read | no |
| `get_flow` | One flow with nodes and edges (`nodeId` for one node). | `flowId` | flows:read | no |
| `summarize_flow_purpose` | One sentence about a flow. | `flowId` | flows:read | no |
| `list_system_templates` | Ready-made automation templates. | `category`, `limit`, `cursor` | flows:read | no |
| `update_flow` | Rename or change description. | `flowId` or `flowName`, `name`, `description` | flows:write | no |
| `update_node` | Change one step. Merges, never replaces. Rejects flows that are on. | `flowName`, `nodeId` (id or label), `text`, `link`, `keywords`, `data`, `name` | flows:write | no |
| `add_node`, `delete_node`, `add_edge`, `delete_edge` | Edit the graph of a draft. | `flowId`, `type`, `data`, `source`, `target`, `nodeId`, `edgeId` | flows:write | no |
| `duplicate_flow` | Copy to a new draft (the way to edit a live flow). | `flowName`, `newName` | flows:write | no |
| `bulk_update_flows` | Edit up to 50 flows at once. | `flowIds`, `filter`, `patch` | flows:write | no |
| `activate_flow` | Turn a flow on. Runs the same checks as the Turn on button: placeholders, plan, keyword conflicts (returned as `warnings`). | `flowName` or `flowId` | flows:write | no, ask the user |
| `deactivate_flow` | Turn a flow off. | `flowName` or `flowId` | flows:write | no |
| `get_flow_revisions`, `revert_flow_to_version` | History and undo for a flow. | `flowId`, `targetVersion` | flows:read / flows:write | no |

## Custom API key only

Not available on one-click Connect keys. Broadcasts, sequences, landing pages, widgets, ref links and tags need an API key carrying the matching scope (OAuth sign-in cannot grant those scopes); `capture_lead` and `resolve` need `accounts:write` (a custom key, or an OAuth client that requests it). Several also need a Starter plan or higher. Do not suggest them unless the user asked for the feature.

| Tool | Purpose | Scope | Confirm |
|---|---|---|---|
| `create_broadcast`, `create_sequence` | Draft a one-off or drip message to contacts. Never sends. | broadcasts:write, sequences:write | yes |
| `create_landing_page`, `create_growth_widget` | Disabled drafts the user enables in SociaHive. | growth_tools:write | yes |
| `create_ref_url` | Tracked link for a flow. | growth_tools:write | no |
| `get_pixel_embed` | Pixel install snippet. | analytics:read | no |
| `search_tags`, `list_tags`, `create_tag`, `find_references` | Contact tags. | tags:read, tags:write | no |
| `capture_lead`, `resolve` | Record a lead; answer a support conversation from the knowledge base. | accounts:write | no |

## Not available to agents

Deleting posts or flows, disconnecting accounts, exporting collected lead values, removing a post's automation, arming a post's automation ("turn on when this post goes live"), Autopilot. If asked, say it is done in the SociaHive dashboard.

## MCP resources

Readable without a tool call: `sociahive://platforms`, `sociahive://trigger-types`, `sociahive://node-types`, `sociahive://post-statuses`, `sociahive://flows`, `sociahive://posts/upcoming`, `sociahive://accounts`.
