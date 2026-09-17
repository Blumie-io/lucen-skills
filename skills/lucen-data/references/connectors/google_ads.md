# Google Ads

## Tables and grain

| Table | Grain | Use for |
|---|---|---|
| `gold_google_ads_customer_performance` | account / day | Account totals |
| `gold_google_ads_campaign_performance` | campaign / day | Cross-channel campaign truth |
| `gold_google_ads_ad_group_performance` | ad group / day | Conventional campaigns. Not Performance Max |
| `gold_google_ads_ad_performance` | ad / day | RSA / conventional ads |
| `gold_google_ads_asset_group_performance` | asset group / day | Performance Max packages |
| `gold_google_ads_device_performance` | account / device / day | Device mix (account grain) |
| `gold_google_ads_network_performance` | account / network / day | Search / YouTube / Discover mix |
| `gold_google_ads_geo_performance` | account / geo / day | Country / presence. Not campaign targeting |
| `gold_google_ads_age_performance` | ad group / age / day | Age diagnostics. Privacy-limited |
| `gold_google_ads_gender_performance` | ad group / gender / day | Gender diagnostics. Privacy-limited |
| `gold_google_ads_keyword_performance` | keyword / day | Search keyword diagnostics |
| `gold_google_ads_search_term_performance` | search term / day | Exposed queries. Partial |
| `gold_google_ads_conversion_action_performance` | conversion action / day | Conversion mix. No spend/clicks |

## Additive / non-additive

Customer and campaign facts are the totals sources. Device / network / geo
are additive **inside their own table** at account grain. Keyword and
search-term tables can be partial — never scale them to account totals. Age
and gender are independently partial.

## Ratios

The server lints `SUM`/`AVG` of these ratios and age × gender joins; the
finding is a caveat, not a block.

Recompute CTR/CPC/CPM from summed components. Do not add `conversions` and
`all_conversions` (alternative measures).

## Do not join / do not sum

**Do not join age × gender.** They are separate facts. Do not treat geo /
device / network as campaign totals. Do not add ad-group **or** asset-group
alone to reconstruct a mixed conventional + Performance Max campaign: sum
both. Conversion-action rows have no impressions, clicks, or spend.

## Attribution

Conversion-action rows carry the **current** attribution and counting
settings, not a historical window column. Do not invent click/view windows.

## Typical questions → table

| Question | Table |
|---|---|
| Account spend / clicks | `gold_google_ads_customer_performance` |
| Campaign ranking | `gold_google_ads_campaign_performance` |
| Performance Max creatives | `gold_google_ads_asset_group_performance` |
| Age or gender mix | the matching table; never both in one query |
