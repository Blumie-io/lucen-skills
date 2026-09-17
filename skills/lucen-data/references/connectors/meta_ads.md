# Meta Ads

## Tables and grain

| Table | Grain | Use for |
|---|---|---|
| `gold_meta_ads_account_performance` | account / day | Account totals, daily trends, **authoritative reach / frequency / unique clicks** |
| `gold_meta_ads_campaign_performance` | account / campaign / day | Campaign delivery, spend, status, objective |
| `gold_meta_ads_adset_performance` | account / ad set / day | Ad set billing event and optimization goal |
| `gold_meta_ads_ad_performance` | account / ad / day | Ad / creative, video, CTA. Gallery grain |
| `gold_meta_ads_action_performance` | ad / day / action type / attribution window / report time | One action type at one window. No delivery columns |
| `gold_meta_ads_placement_performance` | ad / day / publisher / position | Placement delivery |
| `gold_meta_ads_demographic_performance` | account / day / age / gender | Audience mix. Row-level reach only |
| `gold_meta_ads_geo_performance` | account / day / country / region | Geo mix. Row-level reach only |
| `gold_meta_ads_device_performance` | ad / day / device | Device delivery |

## Additive / non-additive

Additive within one table: impressions, clicks, outbound clicks, spend.
`reach`, `frequency`, unique clicks are **account/day only** on
`gold_meta_ads_account_performance`. They are absent on campaign/adset/ad on
purpose. Summing demo or geo reach does **not** reproduce account reach.

## Ratios

The server lints `SUM`/`AVG` of these ratios and of reach/frequency; the
finding is a caveat, not a block.

Recompute `SUM(clicks)/NULLIF(SUM(impressions),0)` (CTR),
`SUM(spend)/NULLIF(SUM(clicks),0)` (CPC),
`SUM(spend)*1000/NULLIF(SUM(impressions),0)` (CPM). Do not AVG `ctr`/`cpc`/`cpm`.

## Do not join / do not sum

Do not join or union gold facts to invent a wider delivery table. Do not sum
action types. Do not sum `cost_per_action`. Demo and geo stay at **account**
grain, not campaign totals.

## Attribution

`gold_meta_ads_action_performance` is keyed by `attribution_window` (e.g.
`7d_click`) and `action_report_time` (`impression` vs conversion time). Filter
to one action type and one window before quoting a number.

## Typical questions → table

| Question | Table |
|---|---|
| How much did we spend / reach this week? | `gold_meta_ads_account_performance` |
| Which campaign spent most? | `gold_meta_ads_campaign_performance` |
| Which ads / creatives? | `gold_meta_ads_ad_performance` |
| Link clicks at 7-day click? | `gold_meta_ads_action_performance` |
| Reach by country? | Say native geo-row reach only; account reach is the account table |
