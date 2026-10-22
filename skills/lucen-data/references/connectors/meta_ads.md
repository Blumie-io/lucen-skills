# Meta Ads

Paid advertising across Facebook, Instagram and other Meta placements. Profile
follower balances use separate social reporting.

## Tables and grain

| Table | Grain | Use for |
|---|---|---|
| `gold_meta_ads_account_performance` | account / day | Account totals, daily trends, **authoritative reach / frequency / unique clicks** |
| `gold_meta_ads_campaign_performance` | account / campaign / day | Campaign delivery, spend, status, objective |
| `gold_meta_ads_adset_performance` | account / ad set / day | Ad set billing event and optimization goal |
| `gold_meta_ads_ad_performance` | account / ad / day | Ad / creative, video, CTA. Gallery grain |
| `gold_meta_ads_action_performance` | ad / day / action type / report time | Six fixed attribution views; conversion values and native purchase ROAS. Context spend repeats across action types |
| `gold_meta_ads_placement_performance` | ad / day / publisher / position | Placement delivery |
| `gold_meta_ads_demographic_performance` | account / day / age / gender | Audience mix. Row-level reach only |
| `gold_meta_ads_geo_performance` | account / day / country / region | Geo mix. Row-level reach only |
| `gold_meta_ads_device_performance` | ad / day / device | Device delivery |

## Additive / non-additive

Additive within one table: impressions, clicks, outbound clicks, spend.
`reach` and `frequency` are native at account/day, campaign/day, and the documented geo/demo row grains. Unique clicks are native at the documented account/demo/geo grains. Never synthesize period reach by summing daily rows. Summing demo or geo reach does **not** reproduce account reach.

## Ratios

The server lints `SUM`/`AVG` of these ratios and of reach/frequency; the
finding is a caveat, not a block.

Recompute `SUM(clicks)/NULLIF(SUM(impressions),0)` (CTR),
`SUM(spend)/NULLIF(SUM(clicks),0)` (CPC),
`SUM(spend)*1000/NULLIF(SUM(impressions),0)` (CPM). Do not AVG `ctr`/`cpc`/`cpm`.

## Do not join / do not sum

Do not join or union gold facts to invent a wider delivery table. Do not sum
action types. Do not sum or average `cost_per_action_*` or native `purchase_roas_*`. Demo and geo stay at **account**
grain, not campaign totals.

## Attribution

`gold_meta_ads_action_performance` has one row per account/ad/day/action type/report time.
Select one action type and metric suffix: `1d_click`, `7d_click`, `28d_click`,
`1d_view`, `1d_ev`, or `meta_default`. Each has `action_count_`, `action_value_`,
`cost_per_action_`, and `purchase_roas_` columns.

`meta_default` is Meta's nested `value` with unified attribution requested and
its reported `attribution_setting` preserved. Do not reconstruct it by adding
windows or assume every ad has the same setting.

Native ROAS comes from `purchase_roas` for `omni_purchase` and
`website_purchase_roas` for `offsite_conversion.fb_pixel_purchase`. Other action
rows and omitted native values have NULL ROAS. Raw conversion values are in
`action_value_*`; do not sum overlapping aliases.

Period ROAS is calculated: aggregate one purchase type's value and divide by
COMPLETE spend from `gold_meta_ads_ad_performance` at matching account/ad/day,
including ad-days without purchase rows. Never use purchase-row spend alone or
average daily native ROAS. Keep currencies separate. Unavailable historical
metrics remain NULL; available counts/costs remain readable.

`instagram_profile_follow` counts ad-attributed Instagram follows on all eight
delivery tables; `instagram_profile_visits` counts ad-attributed visits on
ad/ad set/campaign/placement/device. These scalar outcomes have no per-window
variants. Delivery requests specify no attribution window or unified setting;
do not label these values 1d/7d or equate them with action `meta_default`.
They are not follower balances or net growth. Total Facebook Page
followers use `gold_facebook_page_day.followers` when enabled; other returned
follow/like actions keep their exact action types. Omitted fields stay NULL.
Campaign reach/frequency are native daily metrics and remain nonadditive.

## Typical questions → table

| Question | Table |
|---|---|
| How much did we spend / reach this week? | `gold_meta_ads_account_performance` |
| Which campaign spent most? | `gold_meta_ads_campaign_performance` |
| Which ads / creatives? | `gold_meta_ads_ad_performance` |
| Link clicks at 7-day click? | `gold_meta_ads_action_performance` |
| Reach by country? | Say native geo-row reach only; account reach is the account table |
