# Demo / synthetic gold

Used by bundled templates (`paid-marketing-overview`, `poc-demo`) and orgs
that still have the four-table synthetic paid model. Real pack orgs use
`gold_meta_ads_*` / `gold_google_ads_*` instead — call `get_data_guide` for
those connectors.

## Tables and grain

| Table | Grain | Use for |
|---|---|---|
| `gold_campaign_performance` | account, campaign, day | Spend, funnel, ROAS. The only paid table here with `reach` and `frequency` |
| `gold_ad_performance` | adset, ad, day | Creative drill-down. Additive metrics only |
| `gold_geo_performance` | campaign, country, day | Country or region |
| `gold_demo_performance` | campaign, age, gender, day | Age / gender |
| `gold_social_content` | post | Organic social. Do not union with paid unless asked |

## Additive / non-additive

Additive: spend, impressions, clicks, conversions, conversion_value, units.
Non-additive: `reach`, `frequency` only at campaign grain on
`gold_campaign_performance`. Never SUM them across ads, geos, demos, or
days for unique reach. Reach by country is not in the geo table.

## Ratios

The server lints `SUM`/`AVG` of ratios and of reach/frequency; the finding
is a caveat, not a block.

Recompute `SUM(num)/NULLIF(SUM(den),0)`. Do not AVG or SUM a ratio column.

## Do not join / do not sum

Do not union paid + organic. Demo here is age×gender in one table (unlike
Google Ads / TikTok pack facts).

## Attribution

Synthetic. No platform attribution window.

## Typical questions → table

| Question | Table |
|---|---|
| Campaign ROAS | `gold_campaign_performance` |
| Ad ranking | `gold_ad_performance` |
