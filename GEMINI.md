# SociaHive

This extension connects Gemini CLI to SociaHive: schedule and publish posts on 22 networks (Instagram, TikTok, X, LinkedIn, Facebook, Threads, YouTube, Bluesky, Pinterest, Telegram, Discord, Slack, Mastodon, Tumblr, WordPress and more), build and run DM and comment automations, and read analytics.

Setup:

- The `sociahive` MCP server is at `https://www.sociahive.com/api/mcp`. On first use Gemini CLI opens a browser to sign in. Run `/mcp auth sociahive` to sign in again.
- Call `whoami` first to confirm the connection. MCP, API and CLI access need the 7-day trial or a paid plan; on the free plan every call returns "This SociaHive action was not run". Tell the user to check their plan and stop; do not retry.
- For local files, CSV import and scripts use the CLI: `npm install -g sociahive` (or `npx sociahive`), `export SOCIAHIVE_API_KEY=sk_...`.

Read `skills/sociahive/SKILL.md` before acting; it has the workflows. The tool list is in `skills/sociahive/references/tools.md` and the CLI commands in `skills/sociahive/references/cli.md`.

Core rules:

- Name the platform and let SociaHive pick the account: `platforms: [{"platform":"instagram"}]` (X is `twitter`). When the user names an account (@acme), pass that handle as `accountId`. If SociaHive says it isn't connected or several fit, ask the user which, then retry. Never invent ids, and tell the user the account in `actedOnAccount`.
- Always pass `timezone` (IANA name). `scheduledAt` takes ISO 8601 with an offset or `tomorrow 9am`, `friday 5pm`, `next monday`, `in 2 hours`.
- `create_scheduled_post` without `scheduledAt` saves a draft; with it, the post is scheduled. Prefer scheduling.
- Publishing now (`publish_post_now`) is irreversible: get the user's yes, then repeat the call with the `__confirmToken` from the confirmation response. CLI: `--yes` only after they agree.
- `generate_flow` creates a draft that builds for 10 to 30 seconds. Check it with `get_flow`, show it to the user, and call `activate_flow` only after they say yes. The server does not ask for confirmation there, so you must: it replies to real people.
- Text limits are checked on scheduling: X 280, Bluesky 300, Threads 500, Pinterest 800, Discord 2,000, Instagram and TikTok 2,200, LinkedIn 3,000, Slack 4,000, Telegram 4,096. Use `platforms[].text` for a shorter caption on one network.
- To post in the next free time slot, pass `nextSlot: true` to `create_scheduled_post`. Telegram, Slack, Lemmy, Tumblr, Dev.to, Hashnode, WordPress, Listmonk and Dribbble need a destination: `list_account_destinations`, then `metadata.destination_id`.
- Agents cannot delete posts or flows, disconnect accounts or turn on a post's automation; send the user to the dashboard.
