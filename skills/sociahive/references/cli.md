# sociahive CLI

```bash
npm install -g sociahive        # or: npx sociahive <command>
export SOCIAHIVE_API_KEY=sk_... # Settings, API keys. Needs agent:execute plus the scopes for what you call.
sociahive --help                # every command
sociahive <command> --help      # options of one command
```

`SOCIAHIVE_API_URL` overrides the server origin (the published package defaults to production). The CLI is a thin client over the same tools as MCP, so tool rules, limits and errors are identical.

`--yes` confirms an irreversible step (the CLI otherwise prints the confirmation request and stops). Add it only after the user agreed. For `posts arm`, `flows publish-changes` and `flows discard-changes` the call without `--yes` prints the preview, and `--yes` confirms what it showed.

Commands marked 0.4.0 below need CLI 0.4.0 or later (`npm install -g sociahive@latest`). The `plugs` and `slots` commands also need 0.4.0: in 0.3.1 they fail.

## Commands

```bash
sociahive whoami
sociahive capabilities
sociahive accounts list
sociahive accounts capabilities [--account-id <id>]
sociahive accounts destinations <accountId> [--fresh] [--check "<destination>"]

sociahive upload <file> [<file> ...]          # JPEG, PNG, WebP, MP4, MOV; prints hosted URLs
sociahive posts create (--text "..." | --text-file <path>) \
  --platforms '[{"platform":"instagram"},{"platform":"linkedin","text":"own caption"}]' \
  [--scheduled-at "2026-11-03T09:00:00-05:00"] [--timezone America/New_York] \
  [--media '[{"type":"image","url":"https://..."}]'] [--media-file <path> ...] [--content-type carousel] \
  [--next-slot] [--signature <id|none>] [--alt "..." ...] [--video-cover <url|ms> ...] [--detached <platform> ...]
sociahive posts list [--status draft|scheduled|published|failed|all] [--platform <p>] [--limit <n>]
sociahive posts get <id>
sociahive posts update <id> [--text "..." | --text-file <path>] [--media '[]'] [--scheduled-at "..."]
sociahive posts schedule <id> --at "2026-11-03T09:00:00-05:00" [--timezone <iana>] \
  [--automation-keyword PRICE [--automation-link https://...] | --automation-prompt "..." | --automation-template <id>] \
  [--automation-account <accountId>]
sociahive posts cancel <id>
sociahive posts arm <id> [--yes]              # 0.4.0; the post's comment automation goes live with the post
sociahive posts delete <id> [--yes]           # needs agent:destructive on the key
sociahive posts restore <id>                  # 0.4.0; a deleted post comes back (scheduled ones as drafts)
sociahive posts publish <id> --yes            # irreversible; only after the user agreed
sociahive posts import <file.csv>             # drafts only
sociahive posts duplicate <id>                # copy as a new draft
sociahive posts clicks <id>                   # short-link clicks
sociahive posts review-link <id> [--for "<client name>"] --yes   # Studio, Pro, Agency
sociahive posts review-links <id>

sociahive slots list [--accounts <id,id>]
sociahive slots add --accounts <id,id> --weekday mon --time 09:30 [--timezone Europe/Berlin]
sociahive slots remove <slotId>
sociahive slots next --accounts <id,id> [--after <ISO time>]
sociahive signatures list
sociahive signatures add --name "..." --text "..." [--account <accountId> ...] [--default]
sociahive media list [--kind image|video] [--tag <tag>] [--search <text>]
sociahive media generate (--prompt "..." | --caption "...") [--aspect square|portrait|landscape] [--count 1-4]   # Studio, Pro, Agency
sociahive rss list
sociahive rss pause <feedId>
sociahive rss resume <feedId>
sociahive plugs list <postId>
sociahive plugs add <postId> --metric likes|comments|reposts|views --threshold <n> --action repost|comment [--text "..."] [--account-id <id>] --yes
sociahive clients invite <group id or name> [--platforms instagram,facebook] [--max 5] --yes   # Studio, Pro, Agency
sociahive clients invites [--group <group id or name>]

sociahive flows generate --description "..." [--platform instagram]
sociahive flows bulk-generate --description "..." [--account-id <id>] --yes
sociahive flows list [--status draft|published|all] [--limit <n>] [--query "..."] \
  [--stats [--metric triggers|completions|unique_users|button_clicks]] [--account-id <id or @handle>]   # 0.4.0 flags
sociahive flows get <id>
sociahive flows update <id> [--name "..."] [--description "..."]
sociahive flows duplicate <id> [--name "..."]
sociahive flows activate <id> [--yes]         # turns it on for real people: --yes only after the user agreed
sociahive flows deactivate <id>               # refused while the flow has unpublished changes
sociahive flows update-edge <flow-id> (--edge <id> | --source <step> [--output "No"] [--target <step>]) --to <step>   # 0.4.0
sociahive flows delete-edge <flow-id> (--edge <id> | --source <step> [--output "No"] [--target <step>])              # 0.4.0
sociahive flows publish-changes <id-or-name> [--yes]   # 0.4.0; prints a live flow's unpublished changes, --yes puts them live
sociahive flows discard-changes <id-or-name> [--yes]   # 0.4.0; --yes drops them
sociahive flows delete <id> [--yes]           # needs agent:destructive on the key
sociahive flows restore <id-or-name>          # 0.4.0; back from the trash as a draft
sociahive flows stats <id>
sociahive flows collected-data <id>
sociahive templates list [--category <c>] [--limit <n>]

sociahive analytics [--post-id <id>] [--start-date YYYY-MM-DD] [--end-date YYYY-MM-DD] [--platform <p>]
sociahive insights [--weeks 1-12]
sociahive evergreen list
sociahive evergreen add --text "Caption to recycle"
sociahive ref-url create --name "..." [--flow-id <id>] [--platform instagram] [--message-text "..."] [--plain-code]   # 0.4.0; the last two are WhatsApp only
```

`posts create` needs `--platforms` as a JSON array; `accountId` inside a target is optional (the server picks the account; pass it only to choose between several). Per-target keys: `platform`, `accountId`, `text`, `media`, `contentType` (`carousel` for Instagram), `metadata` (`destination_id` from `accounts destinations`), `article` (Dev.to, Hashnode, WordPress, Listmonk).

## CSV for `posts import`

```csv
text,platforms,media
"Launch day is here",instagram|linkedin,https://example.com/a.jpg
"Behind the scenes",instagram,https://example.com/1.jpg|https://example.com/2.jpg|https://example.com/3.jpg
"Short thought",threads,
```

Header row required with `text` and `platforms`; `media` optional; other columns ignored. Platforms and media URLs are separated by `|`. Media must be public https; 2 to 10 images on an Instagram row make a carousel. Up to 100 rows; a bad row fails alone and is listed with its line number. Every row is a draft; schedule them afterwards.

## Delete commands

`posts delete` and `flows delete` need a custom API key with the `agent:destructive` scope (the one-click Connect key does not have it) and `--yes`. A deleted post is removed from SociaHive only; one already published stays live on the platform. A deleted flow stays in the trash for 30 days. `posts restore` and `flows restore` bring them back. To stop something without deleting it, use `posts cancel` or `flows deactivate`.
