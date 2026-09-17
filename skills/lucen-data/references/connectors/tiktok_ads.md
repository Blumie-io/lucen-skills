# TikTok Ads

## Tables and grain

Prefix `gold_tiktok_ads_`, suffix `_performance`.

| Table | Grain beyond account / day |
|---|---|
| `gold_tiktok_ads_account_performance` | none. Native daily reach / frequency |
| `gold_tiktok_ads_campaign_performance` | campaign |
| `gold_tiktok_ads_ad_group_performance` | campaign + ad group |
| `gold_tiktok_ads_ad_performance` | campaign + ad group + ad |
| `gold_tiktok_ads_conversion_event_performance` | ad + event type. No spend |
| `gold_tiktok_ads_geo_performance` | country |
| `gold_tiktok_ads_age_performance` | age |
| `gold_tiktok_ads_gender_performance` | gender |
| `gold_tiktok_ads_device_performance` | device platform |
| `gold_tiktok_ads_placement_performance` | placement |

## Additive / non-additive

Spend, impressions, destination clicks are additive within one fact and one
currency. Account-day `reach` / `frequency` are **not additive over dates**.
Age and gender are independent; they are not a cube.

## Ratios

The server lints `SUM`/`AVG` of these ratios, reach, and age × gender joins;
the finding is a caveat, not a block.

Recompute CTR/CPC/CPM from summed components (decimal fractions). For
purchase ROAS: filter events to `website_purchase`, aggregate value and
spend separately, join once.

## Do not join / do not sum

Do not join age × gender × geo into an audience cube. Do not sum event types
into total conversions. Event facts have no spend or impressions.

## Attribution

Native attribution is TikTok `standard`. That is **not** a published
click/view window and not conversion-time.

## Typical questions → table

| Question | Table |
|---|---|
| Account spend / reach | `gold_tiktok_ads_account_performance` |
| Campaign ranking | `gold_tiktok_ads_campaign_performance` |
| Website purchases | `gold_tiktok_ads_conversion_event_performance` filtered to `website_purchase` |
