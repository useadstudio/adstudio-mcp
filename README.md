# Adstudio MCP

Cursor / Agent Plugin for [Adstudio](https://useadstudio.com) — ask your connected Google Ads, Meta Ads, and TikTok Ads accounts for performance, structure, and creatives, with provenance on every number.

## Install

1. Install from the Cursor Marketplace (once listed), **or** add the remote MCP URL: `https://useadstudio.com/api/mcp`
2. Sign in to Adstudio and authorize OAuth when prompted
3. Connect ad accounts in Adstudio; access follows your workspace role

## Packages in this repository

| Path | For | Layout |
|---|---|---|
| repository root | Cursor and any [Agent Plugins 1.0.0](https://agent-plugins.org/schemas/1.0.0/plugin.schema.json) host | `plugin.json`, `mcp.json`, `skills/` |
| [`grok/`](grok) | the [xAI plugin marketplace](https://github.com/xai-org/plugin-marketplace) for Grok Build | `.grok-plugin/plugin.json`, `.mcp.json`, `skills/` |

Both describe the same hosted MCP server at `https://useadstudio.com/api/mcp` and carry
the same three skills; only the manifest shape differs, because each host reads its own.

## What it includes

- **MCP server** (`mcp.json`) — hosted at `https://useadstudio.com/api/mcp`
- **Skills**
  - `ad-account-performance-review` — period / period-over-period account summaries
  - `campaign-diagnosis` — root-cause a CPA/ROAS/spend change
  - `creative-audit` — rank ads/creatives and flag fatigue

## Example prompts

- List my ad accounts in Adstudio
- How did my Google Ads campaigns perform last week vs the week before?
- Why did the CPA of my Brand Search campaign go up in the last 14 days?
- Rank the ads in my Summer Sale campaign by ROAS and show what the top three say

## Publish / marketplace

This repo is the Agent Plugin package (`plugin.json` + `mcp.json` + `skills/` + `assets/`) for submission at [cursor.com/marketplace/publish](https://cursor.com/marketplace/publish).

## Links

- Product: https://useadstudio.com
- Support: https://useadstudio.com/support
- Privacy: https://useadstudio.com/privacy-policy
- Terms: https://useadstudio.com/terms-of-service

## License

UNLICENSED — see `plugin.json`.
