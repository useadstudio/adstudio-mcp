---
name: creative-audit
description: Rank the ads and creatives in a Google Ads, Meta Ads or TikTok Ads account, campaign, ad group or ad set by performance, show what each one says and links to, and flag likely creative fatigue. Use when the user asks which ads or creatives are working, which to pause or refresh, whether an ad is fatigued, or wants a creative review.
---

Use this skill when the user wants to know which ads are performing and which are wearing out. Adstudio tools are read-only: you can rank, quote and recommend, never pause or edit an ad.

## Workflow

1. **Discover scope.** Call `list_accounts` (no arguments). If more than one workspace or account could fit, ask which one. Never guess or default ids.
2. **Confirm capabilities.** Call `describe_capabilities` with `workspaceId`, `platform` and `accountId`. Use only the levels, metrics, breakdowns and creative scope levels it returns.
3. **Narrow the scope if asked.** If the user names a campaign, ad group or ad set, resolve its id with `get_entities` (`level: "campaign"`, then `"ad_group"` on Google Ads and TikTok Ads or `"ad_set"` on Meta Ads with `parentId`). Ask when a name matches several entities.
4. **Pull the creatives.** Call `get_creatives` with `scopeLevel` set to `"account"` (omit `entityId`), or `"campaign"`, `"ad_group"` / `"ad_set"`, `"ad"` with the matching `entityId`. `limit` is at most 100; page by narrowing the scope if the response reports `truncated`. Rows contain ad-to-creative assignments and normalized copy, media, destinations and automation flags, but no metrics.
5. **Pull ad performance.** Call `query_performance` with `level: "ad"`, the analysis window (default: the last 30 days, stated explicitly as `YYYY-MM-DD`), `metrics: ["spend", "impressions", "clicks", "conversions", "conversion_value"]` plus `reach`, `frequency`, `ctr`, `cpm`, `video_thruplays` on Meta Ads, `sort: { field: "spend", direction: "desc" }` and `limit` up to 500. Add `filters` with one `{ field: "entity_id", value }` per ad id when the user scoped the audit to a campaign or group. Add a `comparison` window of the same length before it so you can see decay.
6. **Trend for fatigue.** For the top ads by spend, run one more `query_performance` at `level: "ad"` with `breakdowns: ["date"]` and `filters` for those ad ids (weekly is fine: aggregate the daily rows yourself).
7. **Join and compute.** Match performance rows to creative rows by ad id. For each ad compute CTR = clicks / impressions, CPC = spend / clicks, CVR = conversions / clicks, CPA = spend / conversions, ROAS = conversion_value / spend. Compute the same for the comparison window.
8. **Flag fatigue** when an ad shows two or more of the following against its comparison window or its own earlier days: CTR down 20 percent or more with similar or higher impressions, CPA or CPM up 20 percent or more, frequency above roughly 3 on Meta Ads with flat or falling CTR, or a steady week-over-week decline in CTR in the date breakdown. Report the signals you saw, not just the label.

## Output format

- **Ranking table**: ad id and name, campaign, spend, impressions, CTR, CPC, conversions, CPA, ROAS, and a status column (Scale / Keep / Watch / Fatigued / Too little data).
- **What the winners say**: for the top three ads, quote the headline or primary text, describe the media type and the destination from `get_creatives`, and name the common pattern.
- **Fatigue flags**: each flagged ad with the specific signals and numbers behind the flag.
- **Suggestions**: two or three next steps the user can take in their ad platform (refresh a creative, test a new angle, shift attention to a winner). Phrase them as recommendations; the tools cannot apply them.

Use the currency returned by the tool, state the window, and round consistently.

## Rules and pitfalls

- Every call after `list_accounts` names `workspaceId`, `platform` and `accountId` explicitly.
- Ads with fewer than roughly 1,000 impressions or fewer than 10 conversions get "Too little data" rather than a rank by CPA or ROAS.
- Compute rates from raw numbers so Google Ads, Meta Ads and TikTok Ads are comparable; Google rows do not include CTR, CPC or CPM.
- Google Ads has no `frequency` or `reach`; fatigue there rests on CTR and CPA trends plus the date breakdown.
- TikTok Ads media is reported by asset id (Spark Ads by post id); `video_thruplays` is TikTok's 6-second view count, not a full ThruPlay.
- Meta `conversions` and `conversion_value` are attributed action totals and not sortable; sort by `spend` or `clicks`.
- `get_creatives` returns no metrics and `query_performance` returns no copy; always join them, never guess what an ad contains.
- Report `truncated` and `unresolved` fields honestly; say what you could not see.
- Do not promise to pause, edit, duplicate or launch ads; the tools are read-only.
