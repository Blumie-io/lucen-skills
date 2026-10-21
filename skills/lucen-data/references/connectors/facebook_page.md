# Facebook Page (organic)

Organic performance of a Facebook Page: daily Page activity, posts, and reels.
This is not Meta Ads. Paid campaign data lives in the `gold_meta_ads_*` tables.
Reel plays are their own rows. They are never post views.

The pack is staff-only until it has been checked on a real Page. A brand is
filtered by the Page `account_id` on its connection. There is no property map.

## Tables and grain

| Table | Grain | Use for |
|---|---|---|
| `gold_facebook_page_day` | account_id / date | Daily views, organic views, reach, Page views, post engagements, and the follower stock |
| `gold_facebook_page_post` | account_id / post_id | Current posts plus lifetime counters as of the latest snapshot |
| `gold_facebook_page_reel` | account_id / video_id | Current reels plus lifetime plays, likes, comments and shares |

`content_type` on posts and reels uses the shared words `static`, `carousel`,
`video`, `share` and `reels`.

## Additive / non-additive

`views`, `organic_views`, `page_views` and `post_engagements` on
`gold_facebook_page_day` are volumes for that day. Sum them across days.

`reach` is unique viewers for that day. It is a chart value. Do not sum it
into a week or a month.

`followers` is a stock: the last non-null Page follow count on or before the
date. `followers_as_of_date` says which day it came from. Take the latest
value in a period. Never sum daily follower rows.

Post and reel counters are named `cumulative_*`. They are lifetime totals as
of `snapshot_date`. Do not sum them across snapshot dates. The latest snapshot
is the current total. The difference of two snapshots is the change, and this
pack does not precompute that difference. `published_date` is when the content
was published, not the day the views or plays were earned.

Count posts and reels as distinct ids, not as raw insight rows.

NULL means Graph did not return the metric. It is not zero. Do not estimate
organic views as total views minus paid.

## Ratios

No rates are stored. Divide after you aggregate (for example engagements /
views). Do not average a ratio across days or Pages.

## Do not join / do not sum

Do not add `cumulative_play_count` to post views. Reels stay on
`gold_facebook_page_reel`. Do not sum lifetime post counters into
`post_engagements`. Do not sum `cumulative_save_count` (it is always NULL on
Facebook; saves are an Instagram counter). Do not join this connector to
`gold_meta_ads_*` as if both were paid spend, and do not blend it with
Instagram in this pack.

`image_url` and permalinks can expire. They are not copied into storage.
Video and reels prefer the thumbnail. Photos and carousels prefer the full image.

## Attribution

None. These are organic Page interactions, with no attribution window. Page
insights can restate recent days. The steady extract re-fetches the last 7
days. Posts and reels published in the last 90 days are refreshed; older
content is not in v1.

## Typical questions → table

| Question | Table |
|---|---|
| How many Page views and engagements did we get last week? | `gold_facebook_page_day` (sum views and post_engagements; do not sum reach or followers) |
| How many followers does the Page have? | `gold_facebook_page_day` (latest followers, not a sum) |
| What did we publish, and how many lifetime views does each post have? | `gold_facebook_page_post` (latest snapshot; NULL counters mean no insights row) |
| How many plays did our reels get? | `gold_facebook_page_reel` (`cumulative_play_count`, not a post view) |
