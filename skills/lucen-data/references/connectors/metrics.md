# Metrics

Cross-source daily facts built from the ad and store gold tables above. One
run per day, after that day's source syncs finish.

## Tables and grain

| Table | Grain | Use for |
|---|---|---|
| `gold_metrics_account_daily` | source / account / day / currency | Spend, delivery and store revenue per account |
| `gold_metrics_campaign_daily` | source / account / campaign / day / currency | Campaign spend and delivery across ad platforms |
| `gold_metrics_ad_group_daily` | source / account / campaign / ad group / day / currency | Ad group (Meta ad set) spend and delivery |
| `gold_metrics_ad_daily` | source / account / campaign / ad group / ad / day / currency | Ad spend and delivery |

`source` is the connector id: `meta_marketing`, `google_ads`,
`tiktok_marketing`, `linkedin_ads`, `shopify`, `tiendanube`. LinkedIn has no
ad group or ad rows. Shopify and Tienda Nube have account rows only (the
shop or store).

## Additive / non-additive

Every measure (`impressions`, `clicks`, `spend`, `conversions`,
`conversion_value`) is additive across days, accounts and campaigns of the
same source **and the same currency**. `currency` is part of the grain: never
SUM `spend` across currencies. Group by `currency` or filter to one.

A NULL measure means that source does not report it (TikTok and LinkedIn have
no `conversion_value`; Meta is NULL on days or accounts without action data). It is not zero: do not COALESCE it to 0 before an
average or a ratio.

## Ratios

No ratio is stored. Compute it after the sum, from the summed parts, and keep
the source:

- CTR = `SUM(clicks) / SUM(impressions)`
- CPC = `SUM(spend) / SUM(clicks)`
- CPA = `SUM(spend) / SUM(conversions)`
- ROAS = `SUM(conversion_value) / SUM(spend)`

Never AVG a daily ratio: a month of CTR is the month's clicks over the
month's impressions.

## Do not join / do not sum

- `conversions` and `conversion_value` mean different things per source:
  Google Ads conversions, Meta purchases (`omni_purchase`, 7-day click window), TikTok optimization events, LinkedIn website
  conversions, store gross revenue for Shopify and Tienda Nube. Do not add
  them across sources.
- Store revenue (`shopify`, `tiendanube`) and ad platform conversions describe
  the same sales from two sides. Adding them double counts. ROAS from the
  store and ROAS from Meta are two numbers; report both, labelled.
- Do not add a finer grain to a coarser one: account rows already include
  their campaigns.

## Attribution

Each source keeps its own attribution and date convention, copied from its
gold table: ad platforms report by delivery date in the ad account's
timezone, stores by order date in the store's timezone. The metrics tables do
not re-attribute anything.

When a source's sync fails on a day, its rows are left as they were from the
last good run, not set to zero. A recent day can therefore be missing or
stale for one source while the others are current.

## Typical questions → table

| Question | Table |
|---|---|
| Spend by platform this month, per currency | `gold_metrics_account_daily` |
| Which campaigns spent the most across Meta and Google | `gold_metrics_campaign_daily` |
| Monthly CTR of an ad account | `gold_metrics_account_daily` (sum, then divide) |
| Store revenue next to ad spend | `gold_metrics_account_daily`, `shopify` / `tiendanube` rows apart from ad rows |
