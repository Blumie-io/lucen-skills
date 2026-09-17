# Google Business Profile

## Tables and grain

Every row is one Business Profile location per day.

| Table | Grain | Use for |
|---|---|---|
| `gold_google_my_business_location` | location_id / date | Maps and Search impressions, calls, direction requests, website clicks, message conversations |

`account_id` is the parent Business Profile account (`accounts/{id}`).
`location_id` is the location resource (`locations/{id}`). Dates follow the
Performance API calendar, not the org timezone.

## Additive / non-additive

Impression, click, conversation, and direction-request counts are additive
across days for one location. They are **not** a safe account total if you
SUM across locations without grouping: each location is an independent
storefront.

## Ratios

There is no stored CTR or conversion rate on this table. If you derive a
rate, recompute from the counts (for example clicks / impressions), never
AVG a homemade rate.

## Do not join / do not sum

Do not join this table to Google Ads gold on anything other than `date`
(and org identity) to explain local-action conversions. Do not treat a
SUM across locations as the account's Maps presence.

## Attribution

No paid-media attribution window. These are Business Profile interactions
on Google, not ad clicks.

## Typical questions → table

| Question | Table |
|---|---|
| Calls / direction requests / website clicks by location | `gold_google_my_business_location` |
| Maps vs Search impressions | `gold_google_my_business_location` |
