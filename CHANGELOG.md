# Changelog

All notable changes to this repository are listed here. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and versions follow [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

The MCP server itself runs on SociaHive's side and changes without a release here. This file covers the manifests, the skill and the setup docs in this repository.

## [1.1.2] - 2026-10-25

### Added

- The follow check: `generate_flow` and `add_node` take `allowFollowGate` to build an Instagram "Ask to follow" step, so only people who follow the account get the reward. Workflow 5 explains when to use it.

### Removed

- `list_tags` from the tool catalog: the server does not offer it. Use `search_tags`.

## [1.1.1] - 2026-10-25

### Changed

- AI images (`generate_post_image`) cost 5 AI credits each instead of counting against a monthly allowance; running out fails with `credits_exhausted`.
- `upload_media_from_url` also takes `file`: in ChatGPT, an image from the conversation, including one ChatGPT made.

## [1.1.0] - 2026-10-20

### Added

- The skill and tool catalog cover the scheduler tools now open to customers: posting slots (`find_next_slot`, `manage_posting_slots`, `nextSlot`), signatures, the media library, alt text and video covers, duplicate a post, short-link clicks, RSS feeds written by AI, plugs, and on Studio, Pro and Agency AI images, client review links and client invite links.
- The 13 newer networks (Telegram, Discord, Slack, Mastodon, Lemmy, Tumblr, Dev.to, Hashnode, WordPress, Listmonk, Dribbble, Twitch, Kick), with destinations (`list_account_destinations`) and articles.
- Pinterest boards: `list_pinterest_boards` and `metadata.boardId`.
- README links to the Node and Python SDKs and the n8n node.

### Changed

- 22 networks instead of 9 in the descriptions of every manifest and in `server.json` (1.0.2).
- The setup wizard note: `npx @sociahive/mcp add` now covers Codex and Gemini CLI.

## [1.0.0] - 2026-10-18

### Added

- Plugin manifests so SociaHive installs from each tool's plugin system: Claude Code (`.claude-plugin/`), Cursor (`.cursor-plugin/`), Codex (`.codex-plugin/`, `.agents/plugins/marketplace.json`), Grok Build (`.grok-plugin/`) and a Gemini CLI extension (`gemini-extension.json`, `GEMINI.md`).
- The `sociahive` skill (`skills/sociahive/SKILL.md`, copied to the root `SKILL.md`): which MCP tool to use for which job, the CLI commands, the plan requirement and the safety rules.
- Install steps for each tool, `npx skills add sociahive/sociahive-mcp` and `npx @sociahive/mcp add`.

### Changed

- Setup leads with OAuth 2.1 sign-in on first use instead of pasting an API key into a config file. API keys still work.
- The README now covers the plugins, the skill and the CLI, and states that MCP, API and CLI access need the 7-day trial or a paid plan.

## Before 1.0.0 (registry listing, 2026-08-15)

First public version of this repository, before it had a release tag.

### Added

- `server.json`: the MCP registry manifest for `com.sociahive/sociahive`, a streamable HTTP remote at `https://www.sociahive.com/api/mcp`.
- A README with setup for Claude Code, Claude, Cursor and other MCP clients, and the safety model (scoped API keys, confirmation before publishing now, automations created as drafts).

[1.1.0]: https://github.com/sociahive/sociahive-mcp/releases/tag/v1.1.0
[1.0.0]: https://github.com/sociahive/sociahive-mcp/releases/tag/v1.0.0
