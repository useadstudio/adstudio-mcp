---
name: campaign-diagnosis
description: Explain why a Google Ads, Meta Ads or TikTok Ads campaign's CPA, ROAS, conversions or spend changed by decomposing the move into spend, impressions, CTR, CPC and conversion rate and locating it in the campaign's ad groups or ad sets. Use when the user asks "why did my CPA go up", "what happened to campaign X", "why did conversions drop" or wants a root cause for a metric change.
---

Use this skill when the user names a metric that moved and wants to know why. The answer is a decomposition of the change into its drivers, backed by tool rows. Adstudio tools are read-only.

## Workflow

1. **Discover scope.** Call `list_accounts` (no arguments). If more than one workspace or account could fit, ask which one. Never guess or default ids.
2. **Confirm capabilities.** Call `describe_capabilities` with `workspaceId`, `platform` and `accountId` and use only the levels, metrics and breakdowns it lists.
3. **Find the campaign.** If the user gave a name rather than an id, call `get_entities` with `level: "campaign"` (raise `limit` up to 500 if needed) and match the name. If several campaigns match, ask which one.
4. **Set the two windows.** Convert the user's phrasing into `start`/`end` and a `comparison` window of the same length immediately before it, unless the user names both periods. State the windows in the answer.
5. **Campaign-level decomposition.** Call `query_performance` with `level: "campaign"`, `filters: [{ field: "entity_id", value: "<campaignId>" }]`, both windows, and `metrics: ["spend", "impressions", "clicks", "conversions", "conversion_value"]`. Add `search_impression_share`, `search_budget_lost_impression_share` and `search_rank_lost_impression_share` on Google Ads; add `reach`, `frequency`, `cpm` on Meta Ads.
6. **Compute the chain for both windows** and the percentage change of each link:
   - impressions -> CTR (clicks / impressions) -> clicks -> CPC (spend / clicks) -> spend
   - clicks -> CVR (conversions / clicks) -> conversions -> CPA (spend / conversions)
   - conversion_value / spend -> ROAS
   Rank the links by how much they changed. The largest movers are the primary drivers; a CPA rise with flat CPC and falling CVR is a conversion problem, not a traffic-cost problem.
7. **Locate the change one level down.** Call `get_entities` with `level: "ad_group"` (Google Ads, TikTok Ads) or `level: "ad_set"` (Meta Ads) and `parentId: <campaignId>` to get the children's ids. Then call `query_performance` at that level with both windows, `filters` set to one `{ field: "entity_id", value }` entry per child id, and `sort: { field: "spend", direction: "desc" }`. Identify which children account for most of the metric change.
8. **One breakdown if it helps.** With the same filter, run one more `query_performance` using a single breakdown: `date` to see when the change started, `device` on any platform, or `placement`, `age`, `gender` on Meta Ads and TikTok Ads. Only one breakdown per call.
9. **Creative angle if CTR moved.** If CTR is a primary driver, call `get_creatives` with `scopeLevel: "campaign"` and `entityId: <campaignId>` to see what ads and copy are live, and suggest the creative-audit workflow for ranking them.

## Output format

- **Verdict** in two sentences: what moved, by how much, and the main driver.
- **Decomposition table**: metric, previous window, current window, change. Rows: impressions, CTR, clicks, CPC, spend, CVR, conversions, CPA, conversion value, ROAS (only the ones the platform supports).
- **Where it happened**: the two or three ad groups / ad sets (or breakdown segments) that explain most of the change, with numbers.
- **Likely causes** phrased as hypotheses with the evidence for each, followed by what the user could check or change. Distinguish clearly between what the data shows and what you infer.

Use the currency the tool returns, name the windows, and round consistently.

## Rules and pitfalls

- Never default `workspaceId`, `platform` or `accountId`; every call names them explicitly.
- Compute CTR, CPC, CVR, CPA and ROAS yourself from raw rows so both platforms use the same definitions.
- Small conversion counts (roughly under 20 per window) make CPA and CVR changes noisy; say so instead of over-interpreting them.
- `conversions` on Meta Ads are attributed action totals and are not sortable; sort by `spend` or `clicks` instead.
- TikTok Ads does not report `conversion_value` with the `age`, `gender`, `placement` or `device` breakdowns; use `date` or no breakdown for value questions.
- Google `search_*_impression_share` metrics are campaign-level only; `budget_lost` share rising points to a budget cap, `rank_lost` rising points to bids or quality.
- The tools expose no change history, budgets, bids or audiences; say when a suspected cause cannot be verified from the available data.
- Report `truncated` and `unresolved` fields honestly; do not fill gaps with estimates.
- Do not promise or imply that any change can be applied through these tools.
