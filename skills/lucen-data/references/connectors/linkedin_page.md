# LinkedIn Page (organic)

Organic performance of a LinkedIn company Page: followers, page traffic, post
engagement and follower demographics. This is not LinkedIn Ads; paid campaign
data lives in the `gold_linkedin_ads_*` tables.

## Tables and grain

| Table | Grain | Use for |
|---|---|---|
| `gold_linkedin_page_followers` | organization_id / date | Followers gained per day (organic and paid) |
| `gold_linkedin_page_followers_snapshot` | organization_id / snapshot_date | Total followers as of each day we synced (a level, not a gain) |
| `gold_linkedin_page_stats` | organization_id / date | Page views and unique views by page section (overview, about, people, jobs, careers, life, insights, products), desktop and mobile, button clicks |
| `gold_linkedin_page_share_stats` | organization_id / date | Impressions, unique impressions, clicks, likes, comments, shares and engagement for the Page's organic posts, aggregated per day |
| `gold_linkedin_page_follower_demographics` | organization_id / snapshot_date / facet / facet_value | Follower counts by function, industry, seniority, country, region, association type and company size (top 100 values per facet) |
| `gold_linkedin_page_posts` | organization_id / post_urn | The Page's currently published organic posts (the latest sync only; deleted posts disappear) |
| `gold_linkedin_page_post_stats_snapshot` | organization_id / snapshot_date / post_urn | Cumulative per-post counters (`cumulative_impressions`, `cumulative_unique_impressions`, `cumulative_clicks`, `cumulative_likes`, `cumulative_comments`, `cumulative_shares`) as of each sync day |

`organization_id` is the numeric LinkedIn organization id of the Page. One org
can connect several Pages; always keep `organization_id` in the grouping.

## Additive / non-additive

Daily tables (`followers`, `stats`, `share_stats`) are additive across days for
one Page. Snapshot tables are **not** additive across days: `total_followers`,
`follower_count` and every `cumulative_*` per-post counter are values as of
`snapshot_date`. Never add today's follower total to yesterday's. Take the
latest snapshot per Page (and per post) for "as of now", or the difference
between two snapshot dates for change, never a SUM over dates. History starts
on the first collection run and cannot be backfilled before it; for follower
change over the past months use `gold_linkedin_page_followers`. For daily post
activity use `gold_linkedin_page_share_stats`, not the per-post snapshot. `likes` can be negative on a day when reactions are
removed. `shares` excludes instant reposts. `engagement` is a ratio, not a
count.

## Unknown values, provisional days and net change

- **NULL means unknown, not zero.** In `gold_linkedin_page_post_stats_snapshot`,
  `stats_returned` is TRUE when LinkedIn returned numbers for the post. When it is
  FALSE the counters are NULL: the post is older than LinkedIn's 12-month statistics
  window, or LinkedIn gave no answer. Never read those as zero activity.
- **Provisional days.** The daily tables have `is_provisional`: TRUE for the last 2
  days, which LinkedIn may still restate (zeros or partial values). Leave them out of
  week-over-week comparisons.
- **Net follower change.** `gold_linkedin_page_followers_snapshot.net_change` is the
  change in total followers since the previous snapshot. LinkedIn does not expose
  unfollows, so losses can only be implied (gains minus net change) over
  consecutive daily snapshots.
- **Careers views mirror jobs.** `careers_page_views` equals `jobs_page_views`, and
  `all_page_views` already includes jobs: never add careers on top.

## Ratios

`engagement` is provided by LinkedIn as a ratio. Recompute rates from counts
when you aggregate (for example clicks / impressions), never AVG a ratio across
days or Pages.

## Do not join / do not sum

Do not add `follower_count` across facets: each facet breaks down the same
followers, so the facets overlap. `follower_count` is the total of organic and
paid followers. Do not sum per-post stats and expect them to equal
`gold_linkedin_page_share_stats` (daily aggregates and cumulative post counters
are measured differently). Do not mix this connector with `gold_linkedin_ads_*`
as if both were paid spend.

## Attribution

None. These are organic interactions with the Page and its posts, with no
attribution window. Time-bound daily data lags about two days behind today and
the most recent days can be restated.

## Typical questions → table

| Question | Table |
|---|---|
| How many followers did we gain last month? | `gold_linkedin_page_followers` |
| What is our follower total over time? | `gold_linkedin_page_followers_snapshot` |
| How many people visited the Page and which section? | `gold_linkedin_page_stats` |
| How did our posts perform per day? | `gold_linkedin_page_share_stats` |
| Who follows us (industry, seniority, country)? | `gold_linkedin_page_follower_demographics` |
| Which posts are live and what do they say? | `gold_linkedin_page_posts` |
| Which post performs best so far? | `gold_linkedin_page_post_stats_snapshot` (latest snapshot per post) |
