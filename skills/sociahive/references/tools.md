# SociaHive tool catalog

Tools a normal paid or trial account can call over MCP. `scope` is the API key scope; one-click Connect keys and OAuth sign-in carry `agent:execute`, `accounts:read`, `posts:read`, `posts:write`, `flows:read`, `flows:write`, `analytics:read`, `templates:read`, `uploads:write`. Tools needing another scope work only with a custom API key (Settings, API keys).

Ids: `postId` is a scheduler post id; `flowId` a flow id; `accountId` the account's @handle or its `id` from `list_accounts`. Where a tool accepts `postName` or `flowName` (the caption's opening words, the flow's name), use it instead of looking an id up. Rate limits: read 120/min, write 30/min, expensive 5/min, 1,000/day.

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
| `upload_media_from_url` | Copy a JPEG, PNG, WebP, MP4 or MOV into SociaHive from a public URL, or in ChatGPT from a file in the conversation; returns `url`, `mediaType`. 30 per hour. | `url` or `file` (ChatGPT) | uploads:write |
| `list_account_destinations` | Channels, communities, blogs or lists inside one account (Telegram, Slack, Lemmy, Tumblr, Dev.to, Hashnode, WordPress, Listmonk, Dribbble). Pass the id as `metadata.destination_id` on that target; `input` checks one the user named. | `accountId`, `input`, `fresh` | posts:read |
| `list_media` | The user's media library (earlier uploads), newest first. | `kind`, `tag`, `query`, `limit`, `cursor` | posts:read |
| `generate_post_image` | 1 to 4 AI images from a description or the caption. Studio, Pro and Agency; 5 AI credits per image. Too few credits fails with `credits_exhausted`: tell the user how many are needed, do not retry. | `prompt` or `caption`, `aspect`, `count` | uploads:write |
| `list_pinterest_boards` | Boards of each connected Pinterest account. A Pin is saved to a board: pass the chosen board's `id` as `metadata.boardId` on the Pinterest target. An account with one board needs no `boardId`. | `accountId` | posts:read |

The four published-media tools need `accountId` (a @handle or id) when the user has several accounts.

## Posts and scheduling

| Tool | Purpose | Key args | Scope | Confirm |
|---|---|---|---|---|
| `create_scheduled_post` | New post. No `scheduledAt` = draft, with it = scheduled; `nextSlot: true` takes the next free posting slot. | `text`, `platforms[]` (`platform`, optional `accountId`, `contentType` `carousel`, per-target `text`, `media`, `metadata`, `article`, `detached`), `media[]` (`type`, `url`, `altText`, `coverUrl`, `coverTimeMs`), `scheduledAt` or `nextSlot`, `timezone`, `signatureId` | posts:write | no |
| `schedule_post` | Schedule an existing draft; can attach a comment automation (left off). | `postId` or `postName`, `scheduledAt`, `timezone`, `attach_automation` (one of `keyword`+`link`, `prompt`, `template_id`+`inputs`; optional `accountId`) | posts:write (+ flows:write when attaching) | no |
| `arm_post_automation` | Make the comment automation attached to a scheduled post go live with the post. The first call returns a summary and a `contentHash`; after the user agrees, call again with `confirm: true` and that `contentHash`. | `postId` or `postName`, `confirm`, `contentHash` | posts:write, flows:write | yes, two calls |
| `update_scheduled_post` | Edit or reschedule a draft or scheduled post. Media replaces all. | `postId` or `postName`, `text`, `media`, `scheduledAt`, `timezone`, `signatureId`, `detachedPlatforms`, `articles` | posts:write | no |
| `duplicate_scheduled_post` | Copy a post as a new draft (text, media, channels, options, tags). | `postId` or `postName` | posts:write | no |
| `cancel_scheduled_post` | Scheduled to draft. Idempotent. | `postId` or `postName` | posts:write | no |
| `delete_scheduled_post`, `bulk_delete_scheduled_posts` | Delete one post, or up to 50 at once. AI apps: 50 deletes a day. | `postId` or `postName`; `postIds[]` | posts:write, agent:destructive | yes |
| `restore_scheduled_post` | Bring a deleted post back; a scheduled one comes back as a draft. | `postId` or `postName` | posts:write, agent:destructive | yes |
| `bulk_reschedule_posts` | Move up to 50 posts; per-post failures are listed. | `postIds[]`, `scheduledAt`, `timezone` | posts:write | no |
| `publish_post_now` | Publish immediately. Irreversible. | `postId` or `postName` | posts:write | yes |
| `publish` | Same outcome as `publish_post_now`; prefer that one. | `postId`, `platforms` | posts:write | yes |
| `list_scheduled_posts` | Drafts, scheduled, published, failed. | `status` (`draft`, `scheduled`, `published`, `failed`, `all`), `platform`, `limit`, `sortOrder`, `cursor` | posts:read | no |
| `get_scheduled_post` | One post with per-target status and its automation. | `postId` | posts:read | no |
| `get_scheduler_calendar` | Posts in a date range, 200 max. | `fromDate`, `toDate` (ISO), `platform` | posts:read | no |
| `add_to_evergreen`, `list_evergreen` | Queue of recyclable captions that fills gaps in the calendar. | `text`, `sourcePostId` | posts:write / posts:read | no |
| `find_next_slot` | Next free posting slot for these accounts (next 60 days). Read only; holds nothing. | `accountIds`, `after` | posts:read | no |
| `list_posting_slots` | Weekly posting times per account. | `accountIds` | posts:read | no |
| `add_posting_slot`, `update_posting_slot`, `delete_posting_slot` | Add, change or delete a weekly posting time. | `slotId`, `accountIds`, `weekday`, `time`, `timezone` | posts:write | no |
| `list_signatures`, `create_signature` | Saved sign-offs appended when a post publishes; use the id as `signatureId` (`"none"` for no signature). | `name`, `text`, `accountIds`, `isDefault` | posts:read / posts:write | no |
| `list_rss_feeds`, `update_rss_feed` | RSS feeds (up to 5) and how each item becomes a post: draft, AI draft, or AI-written and scheduled into the next slot. Pause, resume or change mode, instructions, tone, accounts, daily cap. | `feedId`, `paused`, `mode`, `instructions`, `tone`, `accountIds` | posts:read / posts:write | no |
| `get_post_link_clicks` | Clicks on a post's short links. | `postId` | posts:read | no |
| `add_post_milestone_action`, `list_post_milestone_actions` | When a post reaches a number of likes, comments, reposts or views on an account, repost it (X) or add a comment (X, Instagram, Facebook), once. Paid plans. | `postId`, `accountId`, `metric`, `threshold`, `action`, `text` | posts:write / posts:read | yes (`add_post_milestone_action`) |
| `create_review_link`, `list_review_links` | Send a post to a client for approval; read what they said. Studio, Pro and Agency. | `postId` or `postName`, `reviewerName` | posts:write / posts:read | yes (`create_review_link`) |
| `create_client_invite`, `list_client_invites` | A link a client opens to connect their own accounts into one of the user's account groups. Studio, Pro and Agency. Needs `accounts:write` (a custom key, or an OAuth client that requests it). | `groupId` or `groupName`, `platforms`, `maxAccounts` | accounts:write / accounts:read | yes (`create_client_invite`) |

A tool the user's plan does not include fails with `feature_disabled`.

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
| `generate_flow` | Build a draft from a description, in the background (10 to 30 s). Expensive bucket. | `description` (10 to 2,000 chars), `platform`, `accountId`, `businessContext`, `variants` (1 to 3), `iterateOnFlowId` (rebuilds a flow the AI made and nobody edited), `allowFollowGate` (Instagram: only people who follow get the reward; pass it only when the user asked) | flows:write | no |
| `create_automation` | Draft from an outcome sentence, returns a plain-language preview. Never goes live. | `outcome`, `accountId` | flows:write | no |
| `generate_multiple_flows` | Rebuild a whole setup (up to 10 drafts), for example migrating from another tool. Starter plan and up. | `description`, `accountId` | flows:write | yes |
| `create_flow` | Empty draft to build node by node. | `name`, `accountId`, `platformUserId`, `platform`, `trigger` | flows:write | no |
| `list_flows` | Browse flows. | `status`, `limit`, `sortBy`, `sortOrder`, `cursor` | flows:read | no |
| `search_flows` | Find a flow by name. | `query`, `limit`, `accountId` | flows:read | no |
| `get_flow` | One flow with nodes and edges (`nodeId` for one node). Each step lists its `outputs` (handle, label, where it leads); each edge has an `id`. | `flowId` | flows:read | no |
| `summarize_flow_purpose` | One sentence about a flow. | `flowId` | flows:read | no |
| `list_system_templates` | Ready-made automation templates. | `category`, `limit`, `cursor` | flows:read | no |
| `update_flow` | Rename or change description. | `flowId` or `flowName`, `name`, `description` | flows:write | no |
| `update_node` | Change one step. Merges, never replaces. Rejects flows that are on. | `flowName`, `nodeId` (id or label), `text`, `link`, `keywords`, `data`, `name` | flows:write | no |
| `add_node`, `delete_node` | Add or remove a step of a draft. `add_node` can add a `follow_gate` (with `allowFollowGate: true`, Instagram only). | `flowId` or `flowName`, `type`, `position`, `data`, `allowFollowGate`, `nodeId` | flows:write | no |
| `add_edge`, `update_edge`, `delete_edge` | Connect, re-route or remove a connection of a draft. Name the output in `sourceHandle` by handle (`button-0`, `yes`, `followed`) or by button label; leave it out for a step's plain next output. One connection per button or branch: use `update_edge` to send it elsewhere. A wrong output is refused with the list of valid ones. | `flowId` or `flowName`, `source`, `target`, `sourceHandle`, `edgeId`, `newTarget` | flows:write | no |
| `duplicate_flow` | Copy to a new draft (edit a live flow while it keeps running). | `flowName`, `newName` | flows:write | no |
| `bulk_update_flows` | Edit up to 50 flows at once. | `flowIds`, `filter`, `patch` | flows:write | no |
| `activate_flow` | Turn a flow on. Runs the same checks as the Turn on button: placeholders, plan, keyword conflicts (returned as `warnings`). | `flowName` or `flowId` | flows:write | no, ask the user |
| `deactivate_flow` | Turn a flow off. | `flowName` or `flowId` | flows:write | no |
| `get_flow_revisions`, `revert_flow_to_version` | History and undo for a flow. | `flowId`, `targetVersion` | flows:read / flows:write | no |
| `delete_flow` | Move a flow to the trash (30 days). | `flowName` or `flowId` | flows:write, agent:destructive | yes |
| `restore_flow` | Bring a flow back from the trash as a draft. | `flowName` or `flowId` | flows:write, agent:destructive | yes |

A live flow can't be edited. To change it in place: `deactivate_flow`, edit, then `activate_flow` (it answers nobody while it is off). To keep it running: `duplicate_flow`, edit the copy, `activate_flow` the copy, then `deactivate_flow` the original. `get_flow` reports `build.status` while a flow is still being built; edits wait until it is `ready`.

## Contacts and growth tools

Broadcasts, sequences, landing pages, widgets, ref links and tags work on Connect keys and OAuth sign-in (an app connected before 2026-10-29 must reconnect to get these scopes). `capture_lead` and `resolve` need `accounts:write` (a custom key, or an OAuth client that requests it). Several also need a Starter plan or higher. Do not suggest them unless the user asked for the feature.

| Tool | Purpose | Scope | Confirm |
|---|---|---|---|
| `create_broadcast`, `create_sequence` | Draft a one-off or drip message to contacts. Never sends. | broadcasts:write, sequences:write | yes |
| `create_landing_page`, `create_growth_widget` | Disabled drafts the user enables in SociaHive. | growth_tools:write | yes |
| `create_ref_url` | Tracked link for a flow. | growth_tools:write | no |
| `get_pixel_embed` | Pixel install snippet. | analytics:read | no |
| `search_tags`, `create_tag`, `find_references` | Contact tags. | tags:read, tags:write | no |
| `capture_lead`, `resolve` | Record a lead; answer a support conversation from the knowledge base. | accounts:write | no |

## Not available to agents

Disconnecting accounts, exporting collected lead values, removing a post's automation, arming a post's automation ("turn on when this post goes live"). If asked, say it is done in the SociaHive dashboard.

## MCP resources

Readable without a tool call: `sociahive://platforms`, `sociahive://trigger-types`, `sociahive://node-types`, `sociahive://post-statuses`, `sociahive://flows`, `sociahive://posts/upcoming`, `sociahive://accounts`.
