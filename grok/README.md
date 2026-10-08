# Adstudio plugin for Grok Build

Connect Grok Build to [Adstudio](https://useadstudio.com) and work with your live ad
accounts from the chat: what was spent, what converted, why a number moved, and which
creatives are carrying the account.

## Installation

In Grok Build, open `/plugin`, search for **Adstudio**, and install.

On first connection, Grok opens Adstudio sign-in in the browser. Authorize with the
Adstudio account whose workspaces Grok should reach. No API key is needed and nothing
should be pasted into chat.

## What you get

- **MCP server** `adstudio` at `https://useadstudio.com/api/mcp` (Streamable HTTP,
  OAuth 2.1).
- **Three skills** that turn the raw tools into whole answers:
  `ad-account-performance-review` (period summary against the previous period),
  `campaign-diagnosis` (decompose a CPA/ROAS/conversion move into its drivers),
  `creative-audit` (rank ads, quote what they say, flag fatigue).

### Tools

Five generic, parameterized tools rather than one per question — the platform is an
argument, so the same tool answers for every account:

| Tool | What it returns |
|---|---|
| `list_accounts` | Every workspace this connection reaches, the ad platforms connected in each, and the selected account per platform |
| `describe_capabilities` | What one account actually supports: levels, metrics, breakdowns, creative scopes |
| `query_performance` | Metric rows at account, campaign, ad group / ad set or ad level, with an optional comparison window and breakdowns |
| `get_entities` | The account's campaigns, ad groups / ad sets and ads with their status and parents |
| `get_creatives` | Ad-to-creative assignments with normalized copy, media, destinations and automation flags |

Platforms the MCP surface reads today: **Google Ads, Meta Ads and TikTok Ads**.
`describe_capabilities` is the live answer for any one account. Adstudio the product
covers more platforms than this surface does; `list_accounts` names what your
connection can reach.

Every tool carries MCP annotations (`readOnlyHint`, `destructiveHint`,
`idempotentHint`, `openWorldHint`), so Grok can tell what a call will do before making
it. Numbers come back with the window and the account they were measured on — an
unresolved input is reported, never silently dropped.

## Example prompts

- "List my ad accounts in Adstudio."
- "How did my Google Ads campaigns do last week versus the week before?"
- "Why did the CPA on my Brand Search campaign go up over the last 14 days?"
- "Which Meta campaigns spent budget without converting this month?"
- "Rank the ads in Summer Sale by ROAS and show me what the top three say."

## Access and control

The access level of an MCP connection is yours to set in Adstudio under
**Settings → MCP** — *Read only*, or *Read and write* — and it is bounded by your own
role in the workspace. Switching the connection off ends all access immediately. The
tools this plugin uses are the measurement tools above.

## Authentication and network

The plugin connects only to `https://useadstudio.com`. Authentication is OAuth 2.1
authorization code with PKCE and dynamic client registration; Grok handles the flow.

Network endpoints:

- `https://useadstudio.com/api/mcp`: hosted MCP (Streamable HTTP)
- `https://useadstudio.com/api/mcp/oauth/authorize`, `/token`, `/register`:
  OAuth 2.1 + DCR
- `https://useadstudio.com/.well-known/oauth-protected-resource/api/mcp`,
  `https://useadstudio.com/.well-known/oauth-authorization-server`: discovery metadata
- `https://useadstudio.com/login`: human sign-in and consent screen during
  authorization

Credentials: an Adstudio account. The MCP server exchanges the grant for a session
scoped to that account and its workspaces, sent as `Authorization: Bearer` on `/mcp`.
No API key, platform token or ad-account credential is stored in the plugin, and the
plugin reads no environment variables, `.env` files or local secrets. It ships no
hooks, commands, agents or install scripts.

## Documentation and support

- Setup guide: https://docs.useadstudio.com/en/features/mcp
- Product: https://useadstudio.com
- Support: https://useadstudio.com/support, contact@useadstudio.com

## License

Proprietary. Use of the hosted MCP server is governed by
[Adstudio's terms](https://useadstudio.com/terms-of-service) and
[privacy policy](https://useadstudio.com/privacy-policy). Adstudio is a trading name of
ENES OZTURK LTD, registered in England and Wales, company number 15234950.
