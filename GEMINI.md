# SociaHive

This extension connects Gemini CLI to SociaHive, a social media platform for scheduling posts on 9 networks (Instagram, TikTok, X, LinkedIn, Facebook, Threads, YouTube, Bluesky, Pinterest), running DM and comment automations, and reading analytics.

- The `sociahive` MCP server is at `https://www.sociahive.com/api/mcp`. On first use Gemini CLI opens a browser to sign in to SociaHive. Run `/mcp auth sociahive` to sign in again.
- The `sociahive` skill (`skills/sociahive/SKILL.md`) lists the tools by job and the CLI commands. Read it before acting.
- For local images and videos, CSV imports or scripts, use the `sociahive` CLI (`npm install -g sociahive`, `export SOCIAHIVE_API_KEY=sk_...`).

Rules:

- Ask the user before publishing to a live audience, turning on an automation, or anything else that cannot be undone. SociaHive asks for a confirmation on those calls; pass it only after the user agrees.
- Automations are created as drafts for the user to review.
- MCP and CLI access need the 7-day trial or a paid plan. On the free plan the tools answer with an upgrade link; show it to the user and stop.
