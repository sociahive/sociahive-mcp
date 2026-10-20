# sociahive CLI

```bash
npm install -g sociahive        # or: npx sociahive <command>
export SOCIAHIVE_API_KEY=sk_... # Settings, API keys. Needs agent:execute plus the scopes for what you call.
sociahive --help                # every command
sociahive <command> --help      # options of one command
```

`SOCIAHIVE_API_URL` overrides the server origin (the published package defaults to production). The CLI is a thin client over the same tools as MCP, so tool rules, limits and errors are identical.

`--yes` confirms an irreversible step (the CLI otherwise prints the confirmation request and stops). Add it only after the user agreed.

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
sociahive flows list [--status draft|published|all] [--limit <n>]
sociahive flows get <id>
sociahive flows update <id> [--name "..."] [--description "..."]
sociahive flows duplicate <id> [--name "..."]
sociahive flows activate <id>                 # turns it on for real people: ask first
sociahive flows deactivate <id>
sociahive flows stats <id>
sociahive flows collected-data <id>
sociahive templates list [--category <c>] [--limit <n>]

sociahive analytics [--post-id <id>] [--start-date YYYY-MM-DD] [--end-date YYYY-MM-DD] [--platform <p>]
sociahive insights [--weeks 1-12]
sociahive evergreen list
sociahive evergreen add --text "Caption to recycle"
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

`posts delete` and `flows delete` exist but need admin-only permissions; for a normal account they fail. Use `posts cancel` and `flows deactivate`, or the dashboard.
