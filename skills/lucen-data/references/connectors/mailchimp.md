# Mailchimp

## Tables and grain

Prefix `gold_mailchimp_`.

| Table | Grain |
|---|---|
| `gold_mailchimp_campaign_performance` | one sent campaign (`account_id`, `campaign_id`) |
| `gold_mailchimp_campaigns_daily` | send day in the account's timezone (`account_id`, `send_date`). A day with no send has no row |
| `gold_mailchimp_audience_growth_daily` | audience + snapshot day (`account_id`, `list_id`, `snapshot_date`) |

`has_report` is false for a sent campaign with no report yet; its `delivered`
and rates are NULL, not zero. A campaign counts as sent when it has a send time
and at least one email out, so a campaign still in `sending` (Timewarp) or
`archived` is included; drafts and scheduled campaigns are not.

`send_date` is the day in the account's own timezone (`account_timezone`), the
day Mailchimp's UI shows, not the UTC day.

## Additive / non-additive

Counts (`emails_sent`, `delivered`, `unsubscribed`, `unique_opens`,
`unique_subscriber_clicks`, `clicks_total`, `ecommerce_total_revenue`, ...) are additive
across campaigns and days. Audience `member_count` is a level, not a flow:
never sum it over dates.

## Ratios

Recompute every rate from summed counts, never average a rate column:
open rate = `SUM(unique_opens) / SUM(delivered)`, click rate =
`SUM(unique_subscriber_clicks) / SUM(delivered)`, click-to-open =
`SUM(unique_subscriber_clicks) / SUM(unique_opens)`.

Clicks: use `unique_subscriber_clicks` (people who clicked). Never use
`unique_clicks` as a numerator: it counts unique clicks per link, so one person
clicking three links adds three, inflating the rate and pushing click-to-open
above 100%.

Opens are inflated by Apple Mail Privacy Protection. `proxy_excluded_unique_opens`
removes those opens, but its denominator (`delivered`) still holds the Apple
recipients, who can never reach the numerator, so
`proxy_excluded_unique_open_rate` is biased low by the audience's Apple Mail
share. Use it to follow one audience over time. To compare audiences with
different Apple shares, use `proxy_excluded_open_rate_reported` (Mailchimp's own
figure) and say that the audiences differ in Apple share.

## Do not join / do not sum

Do not sum `member_count`, `unsubscribe_count` or `cleaned_count` across
dates; use the latest snapshot or `net_member_change`. `net_member_change`
spans `days_since_previous_snapshot` days, so it is not a daily figure when
the pipeline missed days. Audience history starts at the first sync and cannot
be recovered earlier. Do not add revenue across rows with different
`ecommerce_currency_code`.

## Attribution

`ecommerce_*` is Mailchimp's own attribution and needs an e-commerce
integration; it is NULL, not zero, without one. It is not reconciled with
store orders (Shopify, Tienda Nube): the two will not match.

## Typical questions → table

| Question | Table |
|---|---|
| Which campaigns performed best | `gold_mailchimp_campaign_performance` |
| Sends, opens and clicks by day or week | `gold_mailchimp_campaigns_daily` |
| Is the audience growing, and how fast | `gold_mailchimp_audience_growth_daily` |
| Revenue Mailchimp attributes to email | `gold_mailchimp_campaign_performance` (`ecommerce_total_revenue`) |
