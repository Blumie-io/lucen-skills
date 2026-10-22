# Instagram (organic)

Organic performance of an Instagram professional account linked to a Facebook
Page: daily account activity and feed or reel media. This is not Meta Ads.
Stories and ads are not in these tables. There is no reel-play column.

The pack is staff-only until it has been checked on a real account. A brand is
filtered by the Instagram `account_id` on its connection. There is no property map.

## Tables and grain

| Table | Grain | Use for |
|---|---|---|
| `gold_instagram_account_day` | account_id / date | Daily views, reach, profile views, account engagement, and the follower stock |
| `gold_instagram_media` | account_id / media_id | Current feed and reels plus lifetime counters as of the latest snapshot |

`content_type` uses the shared words `static`, `carousel`, `video` and `reels`.

## Additive / non-additive

`views` and `profile_views` on `gold_instagram_account_day` are volumes for that
day. Sum them across days.

`reach` and `accounts_engaged` are unique counts for that day. They are chart
values. Do not sum them into a week or a month. Reach is not an organic-view
substitute.

`total_interactions` is the account-day engagement when Graph returned a daily
figure. Sum it across days. Do not sum media counters to invent it. NULL means
unknown, not zero.

`followers` is a stock: the latest snapshot taken up to the day after the date,
because the run that closes a day runs the next morning.
`followers_snapshot_date` says which day it came from. Take the latest value in
a period. Never sum daily follower rows.

Media counters are named `cumulative_*`. They are lifetime totals as of
`snapshot_date`. Do not sum them across snapshot dates. The latest snapshot is
the current total. The difference of two snapshots is the change, and this pack
does not precompute that difference. `published_date` is when the media was
published, not the day the views were earned.

Count media as distinct ids, not as raw insight rows.

NULL means Graph did not return the metric. It is not zero. Do not estimate
organic as total minus paid.

## Ratios

No rates are stored. Divide after you aggregate (for example interactions /
views). Do not average a ratio across days or accounts.

## Do not join / do not sum

Do not add lifetime media counters into `total_interactions`. Do not treat
`cumulative_save_count` as a Facebook column (saves are populated here). Do not
look for a reel-play column. Stories are not in `gold_instagram_media`. Do not
join this connector to `gold_meta_ads_*` as if both were paid spend, and do not
blend it with Facebook Page in this pack.

`image_url` and permalinks can expire. They are not copied into storage.
Video and reels prefer the thumbnail. Photos and carousels prefer the full image.

## Attribution

None. These are organic account interactions, with no attribution window.
Account insights can restate recent days. The steady extract re-fetches the
last 7 days. Media published in the last 90 days is refreshed; older media is
not in v1.

## Typical questions → table

| Question | Table |
|---|---|
| How many views and profile views did the account get last week? | `gold_instagram_account_day` (sum views and profile_views; do not sum reach or followers) |
| How many followers does the account have? | `gold_instagram_account_day` (latest followers, not a sum; check followers_snapshot_date) |
| What did we publish, and how many lifetime views and saves does each post have? | `gold_instagram_media` (latest snapshot; NULL counters mean no insights row) |
