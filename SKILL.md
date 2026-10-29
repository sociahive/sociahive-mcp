---
name: sociahive
description: Operates a user's SociaHive account through MCP tools or the sociahive CLI. Schedules, drafts, reschedules and publishes posts on 22 networks (Instagram, TikTok, X, LinkedIn, Facebook, Threads, YouTube, Bluesky, Pinterest, Telegram, Discord, Slack, Mastodon, Tumblr, WordPress and more), into weekly posting slots; uploads media; builds and turns on DM and comment automations (comment-to-DM, keyword replies, lead capture) on Instagram, Facebook, Messenger, WhatsApp and Telegram; imports posts from CSV; reads post, automation and content analytics. Use when the user asks to post, schedule or plan social content, attach a comment automation to a post, turn comments into DMs, build an auto-reply or chatbot flow, collect leads, bulk import posts, or check what is working. Needs a SociaHive account on the 7-day trial or a paid plan.
homepage: https://www.sociahive.com/connect-ai
allowed-tools: Bash(sociahive:*)
---

# SociaHive

SociaHive schedules posts on 22 networks, runs DM and comment automations, and reports what works. Drive it with MCP tools when they are connected; use the `sociahive` CLI for local files, CSV import and scripts.

Full tool list: [references/tools.md](references/tools.md). CLI commands and flags: [references/cli.md](references/cli.md).

## Connect and check

1. MCP endpoint: `https://www.sociahive.com/api/mcp` (streamable HTTP). Clients that support OAuth 2.1 (Claude, ChatGPT, Codex, Cursor, Gemini CLI) sign in on first use: the first call returns 401 and a SociaHive sign-in page opens. Others (Grok, custom agents) send an API key (`X-API-Key: sk_...` or `Authorization: Bearer sk_...`).
2. Call `whoami` first. It confirms the connection and returns the tier. If SociaHive tools are not listed, ask the user to connect SociaHive (https://www.sociahive.com/connect-ai) or use the CLI.
3. CLI: `npm install -g sociahive` (or `npx sociahive`), then `export SOCIAHIVE_API_KEY=sk_...` (Settings, API keys; needs `agent:execute`). Check with `sociahive whoami`. Never print the key or write it to files.
4. Plan: MCP, API and CLI need the 7-day trial or a paid plan. On the free plan the tool list still loads, but every call returns an error result starting "This SociaHive action was not run". Nothing ran. Tell the user to check their SociaHive plan and stop; do not retry.

## Which tool for which request

| User wants | Use |
|---|---|
| New post (draft or scheduled) from given text | `create_scheduled_post` |
| Schedule a draft that already exists | `schedule_post` |
| Change time, text or media of an existing post | `update_scheduled_post` (reschedule is always this, never `schedule_post`) |
| Move many posts | `bulk_reschedule_posts` (max 50) |
| Un-schedule a post | `cancel_scheduled_post` (returns it to draft) |
| Post right now | `create_scheduled_post` (no time), then `publish_post_now` after the user agrees |
| What is planned | `list_scheduled_posts`, `get_scheduler_calendar` |
| Next free time ("post it in my next slot") | `create_scheduled_post` with `nextSlot: true`; `find_next_slot` to read it; `list_posting_slots`, `add_posting_slot`, `update_posting_slot`, `delete_posting_slot` to set weekly times |
| Image or video for a post (from a URL, or in ChatGPT an image from the conversation) | `upload_media_from_url`; earlier uploads: `list_media` |
| AI image for a post (Studio, Pro, Agency; 5 AI credits each) | `generate_post_image` |
| Sign-off on posts | `list_signatures`, `create_signature`, then `signatureId` |
| Copy a post | `duplicate_scheduled_post` |
| Client approval or client connects accounts (Studio, Pro, Agency) | `create_review_link`, `list_review_links`; `create_client_invite`, `list_client_invites` |
| Repost or comment once a post takes off (paid plans) | `add_post_milestone_action`, `list_post_milestone_actions` |
| Clicks on a post's links | `get_post_link_clicks` |
| RSS feeds that write posts | `list_rss_feeds`, `update_rss_feed` |
| Local file, CSV, loops | CLI: `sociahive upload`, `posts import` |
| Comment automation tied to a new post | `schedule_post` with `attach_automation` |
| Automation from a description | `generate_flow`, then `activate_flow` after the user agrees |
| Find or edit an existing automation | `list_flows`, `search_flows`, `get_flow`, `update_node` |
| Which reel or post is live on Instagram | `get_latest_published_media`, `list_published_media`, `search_published_media` |
| Performance | `get_analytics_overview`, `get_post_analytics`, `get_content_insights`, `get_flow_stats` |

## Rules that apply to every task

**Confirmation.** Publishing to a live audience cannot be undone. `publish_post_now` (and `publish`, `create_review_link`, `create_client_invite`, `add_post_milestone_action`, `generate_multiple_flows`, `create_broadcast`, `create_sequence`, and the delete and restore tools) first return `{"__mcp_envelope":"pending_confirmation","confirmToken":...,"summary":...}`. Show the user the summary, wait for a clear yes, then call again with the same arguments plus `"__confirmToken": "<token>"`. The token is single use and expires in 5 minutes. In the CLI add `--yes` only after the user agreed. `activate_flow` has no confirmation step, so you must ask first: it makes the automation reply to real people.

**Platform ids** for `platforms[].platform` and filters: `instagram`, `facebook`, `twitter` (this is X), `linkedin`, `threads`, `tiktok`, `youtube`, `bluesky`, `pinterest`, `telegram`, `discord`, `slack`, `mastodon`, `lemmy`, `tumblr`, `devto`, `hashnode`, `wordpress`, `listmonk`, `dribbble`, `twitch`, `kick`. Flows use only `instagram`, `whatsapp`, `messenger`, `telegram`.

**Destinations and articles.** Telegram, Slack, Lemmy, Tumblr, Dev.to, Hashnode, WordPress, Listmonk and Dribbble post to a channel, community, blog or list inside the account: get its id with `list_account_destinations {"accountId":"..."}` (`input` checks one the user named) and set `metadata.destination_id` on that target. Dev.to, Hashnode, WordPress and Listmonk are articles: pass `article` (`title`, Markdown `body`, optional `canonicalUrl`, `coverImage`, `tags`) on the target. The server says which part is missing.

**Accounts.** Name the platform and let the server pick the account: `platforms: [{"platform":"instagram"}]`. Do not call `list_accounts` first. When the user names an account ("on @acme"), pass that handle as `accountId` to every tool that takes one, including `platforms[].accountId`; the `id` from `list_accounts` works too. If the named account isn't connected, or several accounts fit, the tool changes nothing and returns the connected accounts: tell the user, ask which one, then retry with their choice. Never act on a different account than the one the user named without asking, and never invent an id. Results include `actedOnAccount`: name that account to the user, not one from their message.

**Time.** `scheduledAt` takes ISO 8601 with an offset (`2026-11-03T09:00:00-05:00`) or a short phrase the server resolves: `tomorrow 9am`, `friday 5pm`, `next monday`, `in 2 hours`. A day without a time means 09:00. Always send `timezone` as an IANA name (`America/New_York`); ask the user if you do not know it, because without it the server assumes UTC. A time in the past is rejected. When the phrase could mean two dates (for example "saturday" on a Saturday, or "next tuesday" on a Sunday) or the user's date matters, compute the ISO time with the right offset yourself and read back the date the server resolved. In the CLI always use ISO.

**Draft, scheduled, now.** `create_scheduled_post` without `scheduledAt` saves a draft; with it the post is scheduled. There is no create-and-publish call. Prefer scheduling; publish now only when the user says now. A blanket "just do it, don't ask me anything" does not replace the confirmation: still show the one-line summary (text, networks) and get a yes before the `__confirmToken` call. If the user's text is over a network's limit, show the shortened version for that target before using it; never cut their words silently.

**Limits are checked when a post is scheduled or published, not when a draft is saved.** Text per target: X 280, Bluesky 300, Threads, Twitch and Kick 500, Pinterest 800, Discord 2,000, Instagram and TikTok 2,200, LinkedIn 3,000, Slack 4,000, Telegram 4,096 (1,024 with media), YouTube 5,000, Facebook 63,206. Use `platforms[].text` for a shorter caption on one network instead of cutting the main text.

**Media.** `media: [{"type":"image"|"video","url":"https://..."}]`, public https URLs or URLs from `upload_media_from_url` or `sociahive upload`. Add `altText` to an image, and `coverUrl` (or `coverTimeMs`) to a video for its cover. Instagram posts need media (ask for it before drafting one; scheduling fails without it); a carousel is `contentType: "carousel"` on the Instagram target with 2 to 10 items. X takes up to 4 images or 1 video, not both. Bluesky 4 images or 1 video. YouTube needs a video (first line of the text is the title). TikTok needs media. Pinterest needs an image and a board: call `list_pinterest_boards` and set `metadata.boardId` on the Pinterest target (not needed when the account has only one board). Instagram Reels and Stories are not available to normal accounts yet: do not set `contentType` to `reels` or `stories`. X is a paid-plan feature. A feature the user's plan does not include fails with `feature_disabled`: tell the user which plan has it; do not retry.

**Deleting.** The agent cannot delete posts or flows or disconnect accounts. Offer `cancel_scheduled_post` or `deactivate_flow`, or send the user to the dashboard.

**Rate limits.** 120 reads, 30 writes and 5 expensive calls (`generate_flow`, `search_published_media`) per minute, 1,000 calls per day. On `rate_limit`, wait `Retry-After` seconds. Use the bulk tools instead of loops.

## Workflows

### 1. Schedule a post to several networks

"Post this to Instagram and LinkedIn next Tuesday at 9am" (image at a URL, user is in New York).
1. If the image URL is not already public https or you want it hosted: `upload_media_from_url {"url":"https://..."}` returns `url` and `mediaType`. In ChatGPT, an image in the conversation (one ChatGPT made or the user attached) goes in as `upload_media_from_url {"file": <that image>}`; ChatGPT fills in the file object.
2. `create_scheduled_post {"text":"...","media":[{"type":"image","url":"<url>"}],"platforms":[{"platform":"instagram"},{"platform":"linkedin","text":"LinkedIn version"}],"scheduledAt":"next tuesday 9am","timezone":"America/New_York"}`.
3. Read back `status: "scheduled"` and `scheduledAt`, and tell the user the local time. Report per-target errors (`TEXT_TOO_LONG` names the target) and fix only that target.

### 2. Publish now

1. `create_scheduled_post` without `scheduledAt` (a draft). Show the user the text, media and networks.
2. After a clear yes: `publish_post_now {"postId":"<id>"}`. It returns `pending_confirmation`; show the summary, get a second yes if the first was general, then repeat the call with `__confirmToken`.
3. Status becomes `publishing`. Tell the user it cannot be undone from here.

### 3. Move, edit or cancel posts

- One post: `update_scheduled_post {"postName":"launch announcement","scheduledAt":"friday 5pm","timezone":"Europe/London"}` (or `postId`). Published or publishing posts are rejected.
- Many posts: `list_scheduled_posts {"status":"scheduled"}` then `bulk_reschedule_posts {"postIds":[...],"scheduledAt":"monday 9am","timezone":"..."}`.
- Cancel: `cancel_scheduled_post {"postId":"..."}`.
- See the week: `get_scheduler_calendar {"fromDate":"2026-11-02T00:00:00-05:00","toDate":"2026-11-08T23:59:59-05:00"}` (200 posts max). The range has no timezone argument, so give ISO times with the user's offset; a bare date is read as UTC and can miss late-evening posts. Filter on `status` yourself.

### 4. Turn a new post's comments into DMs

Instagram feed, carousel or reel posts only.
1. `create_scheduled_post` as a draft with `platforms: [{"platform":"instagram"}]` (no `scheduledAt`).
2. `schedule_post {"postId":"<id>","scheduledAt":"tomorrow 9am","timezone":"...","attach_automation":{"keyword":"PRICE","link":"https://example.com/price"}}`. Pick exactly one door inside `attach_automation`: `keyword` (+ optional `link`), `prompt` (plain description), or `template_id` (+ `inputs`). Add `accountId` inside it only if the post targets several Instagram accounts. Needs the `flows:write` scope.
3. The automation is created attached to the post and turned OFF (`armed: false`). Tell the user to review and switch it on from the post's page in the SociaHive scheduler (web or iOS); an agent cannot arm it.

For a post that is already live, use workflow 5 and describe which post it listens on.

### 5. Build an automation and turn it on

1. `generate_flow {"description":"When someone comments PRICE on my latest reel, reply 'check your DMs' and DM my price list https://example.com/price","platform":"instagram","accountId":"<the @handle the user named, if any>"}`. Description is 10 to 2,000 characters: trigger plus outcome. Optional `businessContext` (`industry`, `product`, `keyword`, `offer`, `tone`). Flow platforms are `instagram`, `whatsapp`, `messenger`, `telegram`. To tie it to a live post find it first (`get_latest_published_media`) and name it in the description.
2. It builds a draft in the background, 10 to 30 seconds. Poll `get_flow {"flowId":"<id>"}` until it has nodes. Never report it as done before that.
3. Show the user the trigger, keyword and each message in plain words. Fix details with `update_node {"flowName":"...","nodeId":"the welcome message","text":"..."}` (also `link`, `keywords`). A `[link]` style placeholder left in the copy blocks turning it on.
4. Ask: "Turn this on? It will reply to real people." Even if the user said "turn it on right away" earlier, they have not seen the built content yet: show it, ask once, and only after that yes: `activate_flow {"flowName":"..."}`. Relay `warnings` of type `keyword_conflict`: another live flow on the account already uses that keyword.
5. Editing a flow that is on returns `state_error`. For a small change, after a yes: `deactivate_flow`, edit, check with `get_flow`, then `activate_flow` (the flow answers nobody while it is off). To keep it answering while you work: tell the user the plan, then `duplicate_flow {"flowName":"Summer Promo","newName":"Summer Promo (SALE)"}`, `get_flow` on the copy to read the trigger node's id or label, `update_node {"flowName":"Summer Promo (SALE)","nodeId":"<trigger label>","keywords":["SALE"]}` (`keywords` replaces the list), check with `get_flow`, `activate_flow` the copy after a yes, then `deactivate_flow` the original after a yes. If the keyword is unchanged, deactivate the original first so two flows do not answer the same comment.

Instagram rules to keep when you edit by hand: in a comment-to-DM flow the first DM is one message only, and Instagram only lets a business message someone inside its 24 hour messaging window (a comment opens it).

**Follow check ("Ask to follow").** If the user wants the link or reward only for people who follow the account, pass `"allowFollowGate": true` to `generate_flow` and say so in the description ("only send the link to people who follow me"). It works on Instagram only. The flow asks the person to follow and tap "I followed", Instagram confirms the follow, and only then does the `followed` branch run. Add it only when the user asked for it: Instagram discourages pushing people to follow. People who have never messaged the account can't be checked before that first tap; that's Instagram's rule, not a bug. To add one to an existing Instagram draft: `add_node {"flowId":"<id>","type":"follow_gate","position":{"x":0,"y":0},"allowFollowGate":true}`, then connect each output once: `add_edge {"flowId":"<id>","source":"<the new node id>","target":"<the step that gives the reward>","sourceHandle":"followed"}`, and the same with `"sourceHandle":"not_followed"` for people who don't follow. After a comment trigger the follow step must come right after the trigger. Its messages and number of tries are the product defaults; the user changes them in the flow editor.

### 6. Find what works and plan next week

1. `get_content_insights {"weeks":8}` (1 to 12): which formats, weekdays and pillars beat the account's own average, with sample sizes. Ignore small samples and say when data is insufficient.
2. `get_analytics_overview {"startDate":"2026-10-01","endDate":"2026-10-28"}` (YYYY-MM-DD; optional `platform`) and `get_post_analytics {"postId":"..."}` for single posts.
3. `get_scheduler_calendar` for next week to avoid clashes.
4. Propose the plan as a short list (day, network, topic, format) with the reason from the data. Do not invent numbers.
5. Create the posts only after the user approves the list, as drafts or scheduled (workflow 1).

### 7. Bulk import posts from a CSV (CLI)

`sociahive posts import posts.csv`. Header row required: `text` and `platforms` (separate several with `|`, for example `instagram|linkedin`), optional `media` (public https URLs separated by `|`; 2 to 10 images on an Instagram row make a carousel). Up to 100 rows. Every row becomes a draft; nothing is scheduled. There is no date column (extra columns such as `date` are ignored, so keep the dates yourself). A file with other column names or comma-separated networks must be converted first (`caption` to `text`, `instagram, linkedin` to `instagram|linkedin`). The result lists `created`, `failed`, `post_ids` and per-row `errors` by line number. Schedule afterwards, one call per draft id with `schedule_post` or `sociahive posts schedule <id> --at ... --timezone ...`.

### 8. Local image or video, then post

1. `sociahive upload ./photo.jpg ./clip.mp4` prints hosted URLs (JPEG, PNG, WebP, MP4, MOV up to the plan's size cap).
2. Use the URLs in `create_scheduled_post` media, or in one command: `sociahive posts create --text "..." --platforms '[{"platform":"instagram"}]' --media-file ./photo.jpg --scheduled-at "2026-11-03T09:00:00-05:00" --timezone America/New_York`.

## Errors

| Error | Meaning and action |
|---|---|
| Result starts "This SociaHive action was not run" | Plan has no AI app access. Tell the user; stop. |
| `auth_error` | Key or sign-in invalid, revoked, or missing a scope (`missingScope`). Reconnect, or mint a key with that scope. |
| `ambiguous_reference` | Several accounts, posts or flows match. Ask the user which; pass the id. |
| `no_account_selected` | No account fits. Ask the user which account; pass its @handle or id as `accountId`. |
| `validation_error` on `accountId` | The account named isn't connected, or several fit. The message lists the connected accounts: ask the user, then retry with their choice. |
| `not_found` | Wrong id or not the user's. Look it up by name or list; do not guess. |
| `validation_error` with `TEXT_TOO_LONG` | The message names the target. Shorten that target with `platforms[].text`. |
| `validation_error` with `OPTION_UNAVAILABLE` | Account lacks a permission or the plan lacks the platform (X). Tell the user; they may need to reconnect the account. |
| other `validation_error` | Read the message: past time, unfilled placeholder, flow cannot go live. Fix the input. |
| `state_error` | Flow is on, or the post is publishing or published. Duplicate the flow, or wait. |
| `tier_error`, `limit_reached`, `automated_account_limit` | Plan limit. Tell the user which; do not retry. |
| `rate_limit` | Wait `Retry-After`, then continue. |
| `upstream_error` | Platform hiccup. Retry once or twice with a pause; if it persists, report it. |
| Account status `needs_reauth` | The user must reconnect it in SociaHive. Posting to it will fail. |

## Never

- Publish, turn on an automation or send anything to real people without the user's explicit yes in this conversation.
- Invent account, post or flow ids, platform limits or analytics numbers.
- Retry after a plan error or loop on a failing call.
- Print or store the API key.

Links: https://www.sociahive.com/connect-ai (connect guides), https://www.sociahive.com/developers (docs), https://www.sociahive.com/settings?tab=api-keys (keys).
