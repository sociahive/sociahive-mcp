# Publishing

How this repository gets listed in each agent's marketplace. Each step is done by a person with the SociaHive accounts; nothing here publishes on its own.

## Before anything

1. Create the public GitHub repository `sociahive/sociahive-mcp` and push this folder to `main`.
2. Check that the manifests still match the live server: endpoint `https://www.sociahive.com/api/mcp`, and the tool names in `skills/sociahive/SKILL.md` against what `/api/mcp` lists.
3. Bump `version` in every manifest together when something changes: `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `.cursor-plugin/plugin.json`, `.cursor-plugin/marketplace.json`, `.codex-plugin/plugin.json`, `.grok-plugin/plugin.json`, `.grok-plugin/marketplace.json`, `gemini-extension.json`.

## Claude Code

- Works as soon as the repo is public: `/plugin marketplace add sociahive/sociahive-mcp`.
- Validate first: `claude plugin validate --strict .`
- Official directory: submit through Anthropic's plugin directory form with the repo URL.

## Codex

- Works as soon as the repo is public: `codex plugin marketplace add sociahive/sociahive-mcp` (reads `.agents/plugins/marketplace.json`).
- Official plugin directory: submit through the OpenAI developer dashboard. The `interface` block in `.codex-plugin/plugin.json` is the listing text.

## Gemini CLI

- Works as soon as the repo is public: `gemini extensions install https://github.com/sociahive/sociahive-mcp`.
- Gallery: add the GitHub topic `gemini-cli-extension` to the repo; the gallery picks up public repos with that topic and a root `gemini-extension.json`.

## Cursor

- Submit the repo at the Cursor marketplace (cursor.com/marketplace, "Submit a plugin"). It reads `.cursor-plugin/plugin.json` and `.cursor-plugin/marketplace.json`.

## Grok Build

- Works as soon as the repo is public through `.grok-plugin/marketplace.json`. Submit to the xAI plugin directory when it accepts outside listings.

## skills.sh

- Listed automatically once someone installs it: `npx skills add sociahive/sociahive-mcp`. The skill is `skills/sociahive/SKILL.md`; the root `SKILL.md` is a copy for tools that read only the root. Keep the two identical.

## After a change to the MCP tools

Update the tool list in `skills/sociahive/SKILL.md`, copy it to the root `SKILL.md`, bump the versions, push.
