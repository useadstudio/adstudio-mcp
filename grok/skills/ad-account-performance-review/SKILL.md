---
name: ad-account-performance-review
description: Summarize how a Google Ads, Meta Ads or TikTok Ads account performed over a period (last week, last month, a custom date range) and compare it with the previous period. Use when the user asks for a performance review, weekly or monthly report, "how did my ads do", or a period-over-period comparison of spend, conversions, CPA or ROAS.
---

Use this skill when the user wants a period summary of one or more connected ad accounts. Adstudio tools are read-only: you can report and interpret, never change anything.

## Workflow

1. **Discover scope.** Call `list_accounts` (no arguments). It returns every workspace this connection reaches, the platforms connected in each (`google_ads`, `meta_ads`, `tiktok_ads`) and the selected account per platform.
   - If more than one workspace or account could fit, ask the user which one before going further. Never guess or default an id.
   - If the user says "all my ads" and the workspace has more than one ad platform connected, review each platform separately (steps 2-4 per platform) and then combine.
2. **Confirm what is queryable.** Call `describe_capabilities` with `workspaceId`, `platform` and `accountId`. Use the returned levels, metrics and breakdowns; do not request anything it does not list.
3. **Resolve the dates.** Convert the user's phrase into explicit `YYYY-MM-DD` ranges. "Last week" = the last full Monday-Sunday week; "last month" = the last full calendar month. Set `comparison` to the immediately preceding period of the same length unless the user names another baseline. State the ranges you chose in the answer.
4. **Query account totals.** Call `query_performance` with:
   - `level: "account"`, `start`, `end`, `comparison: { start, end }`
   - `metrics`: `["spend", "impressions", "clicks", "conversions", "conversion_value"]`, plus platform extras when useful (Google campaign level: `search_impression_share`, `search_budget_lost_impression_share`, `search_rank_lost_impression_share`; Meta: `reach`, `frequency`, `cpm`, `ctr`, `cpc`).
5. **Query campaign detail.** Call `query_performance` again with `level: "campaign"`, the same dates and comparison, `sort: { field: "spend", direction: "desc" }` and `limit: 50`. This is what the top movers section is built from.
6. **Optional trend.** If the user asks how the period developed, run one more `query_performance` at `level: "account"` with `breakdowns: ["date"]`. Only one breakdown is allowed per call.
7. **Compute derived metrics yourself** from the raw rows, for both the current and the comparison period:
   - CTR = clicks / impressions, CPC = spend / clicks, CVR = conversions / clicks
   - CPA = spend / conversions, ROAS = conversion_value / spend
   Google Ads rows do not include CTR, CPC or CPM; Meta and TikTok rows may, but compute them from the raw numbers anyway so the platforms are comparable.

## Output format

Lead with a two-to-three sentence verdict, then:

- **Totals table**: metric, current period, previous period, absolute and percentage change. Include spend, impressions, clicks, CTR, CPC, conversions, CVR, CPA, conversion value, ROAS.
- **Top movers**: up to five campaigns whose spend or CPA changed the most, with the numbers.
- **What to look at next**: two or three concrete follow-up questions the user can ask (for example, "diagnose campaign X" or "audit the creatives in campaign Y").

Show the currency exactly as the tool returns it and say which date ranges were used. Round money to two decimals and rates to one decimal place.

## Rules and pitfalls

- Always start with `list_accounts`; every other call needs an explicit `workspaceId`, `platform` and `accountId`.
- Metrics differ by platform and level; only request what `describe_capabilities` listed for that level.
- Meta `conversions` and `conversion_value` are attributed action totals; say "attributed conversions" rather than presenting them as sales.
- Conversion counts near zero make CPA and ROAS unstable. Flag that instead of ranking by them.
- If a response reports `truncated`, say the list is partial and offer to narrow the range or raise `limit` (max 500).
- If the envelope lists `unresolved` items, mention what could not be answered and why. Do not fill gaps with estimates.
- Do not compare rows from different platforms as if they were one dataset; present them side by side and note attribution differs.
- Never promise budget changes, pausing, bid edits or any action; the tools cannot perform them.
