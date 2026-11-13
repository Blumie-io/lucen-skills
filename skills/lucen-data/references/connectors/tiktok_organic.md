# TikTok (organic)

Organic account counters of a TikTok account that connected through Login Kit,
read through the Display API. This is not TikTok Ads (`gold_tiktok_ads_*`): a
different app, connector and bronze dataset. A brand is filtered by the TikTok
`open_id` on its connection. There is no property map.

## Tables and grain

| Table | Grain | Use for |
|---|---|---|
| `gold_tiktok_organic_account_day` | open_id / snapshot_date | Followers, following, lifetime likes and public video count as read each day, and their change since the previous snapshot |
| `gold_tiktok_organic_video` | open_id / video_id | The account's current public videos with lifetime views, likes, comments and shares, and their change since the video's previous snapshot |

## Additive / non-additive

TikTok exposes lifetime totals only and no history, so this pack reads them once
per run and history accrues from our own runs. Every `cumulative_*` column is a
stock as of `snapshot_date`: never sum it across days. Take the latest value in a
period.

`follower_change`, `following_change`, `likes_change` and `video_change` are the
difference between a snapshot and the previous one. They can be negative (an
unfollow, a removed video). They are NULL on the first snapshot. After a missed
run a change spans every day since `previous_snapshot_date`, so do not read it as
one day. Sum a change over a period only when `previous_snapshot_date` shows no
gap, or take the difference of the first and last `cumulative_*` values instead.

`gold_tiktok_organic_video` counters are `cumulative_*`: lifetime totals as of `snapshot_date`. Do not sum them across days. Its table is the current state, not history: only public videos from the latest completed run appear, so a deleted or now-private video disappears. Count videos as distinct `video_id`. `published_date` is when the video was created, not when its views were earned. `view_change` and the other `*_change` columns compare against the video's previous snapshot and are NULL the first time a video is seen.

NULL means TikTok did not return the field. It is not zero.

## Ratios

No rates are stored. Divide after you aggregate. Do not average a ratio across
days.

## Do not join / do not sum

Do not sum `cumulative_*` columns across days. Do not join this connector to
`gold_tiktok_ads_*` as if both were paid spend, and do not blend it with
Instagram or Facebook Page in this pack. Watch time, reach and audience
breakdowns are not available from this API.

## Attribution

None. These are organic account counters with no attribution window. Counters
are read once per daily run, so a day with no run has no row.

## Typical questions → table

| Question | Table |
|---|---|
| How many followers does the account have? | `gold_tiktok_organic_account_day` (latest `cumulative_follower_count`, not a sum) |
| How many followers did we gain last week? | `gold_tiktok_organic_account_day` (last minus first `cumulative_follower_count`; check `previous_snapshot_date` for gaps) |
| Which videos perform best? | `gold_tiktok_organic_video` (rank by `cumulative_view_count`; compare videos by `published_date`) |
| How many videos has the account published? | `gold_tiktok_organic_account_day` (latest `cumulative_video_count`) |
