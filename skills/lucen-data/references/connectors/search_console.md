# Search Console

## Tables and grain

Every row is scoped by `site_url`, `date`, `search_type`. Dates use Google's
Pacific calendar.

| Table | Additional grain | Use for |
|---|---|---|
| `gold_search_console_traffic` | none | Site clicks, CTR, position trends |
| `gold_search_console_page` | page_url | Pages and hostnames |
| `gold_search_console_query` | query | Disclosed search terms |
| `gold_search_console_device` | device | Mobile / desktop / tablet |
| `gold_search_console_country` | country_code | Geo |
| `gold_search_console_page_query` | page_url, query | Queries on a page |

## Additive / non-additive

`clicks` and `impressions` are additive **within one table**. Site clicks
(traffic) and page clicks are different measures — do not add them.

## Ratios

The server lints `SUM`/`AVG` of `ctr` and `average_position`; the finding
is a caveat, not a block.

CTR is a fraction. Never `SUM(ctr)` or `AVG(average_position)`. Recompute:

```sql
SUM(clicks) / NULLIF(SUM(impressions), 0) AS ctr
SUM(position_impression_sum) / NULLIF(SUM(impressions), 0) AS average_position
```

## Do not join / do not sum

Do not add page rows to reconstruct site totals. Do not add Domain +
URL-prefix properties. Page/query is a native table, not a join you invent.

## Attribution

No paid-media attribution window. `search_type` is the scope (this ship is
finalized Web search).

## Typical questions → table

| Question | Table |
|---|---|
| Site clicks / impressions | `gold_search_console_traffic` |
| Top pages | `gold_search_console_page` |
| Queries for a page | `gold_search_console_page_query` |
