# Google Analytics (GA4)

## Tables and grain

Prefix `gold_google_analytics_`. Every performance fact has a matching
`<grain>_events_daily` and `<grain>_key_event_pivot_daily`.

| Table | Grain after property / date |
|---|---|
| `gold_google_analytics_property_daily` | Property totals (use this for headlines) |
| `gold_google_analytics_landing_page_daily` | Session entry page |
| `gold_google_analytics_page_daily` | Page path |
| `gold_google_analytics_geography_daily` | Country / region / city |
| `gold_google_analytics_session_channel_daily` | Session default channel |
| `gold_google_analytics_first_user_channel_daily` | First-user channel |
| `gold_google_analytics_session_source_daily` | Session source |
| `gold_google_analytics_session_source_medium_daily` | Session source / medium |
| `gold_google_analytics_session_source_medium_content_daily` | Session source / medium / content |
| `gold_google_analytics_session_manual_source_daily` | Manual session source |
| `gold_google_analytics_session_manual_source_medium_daily` | Manual session source / medium |
| `gold_google_analytics_session_manual_source_medium_content_daily` | Manual session source / medium / content |
| `gold_google_analytics_first_user_source_daily` | First-user source |
| `gold_google_analytics_first_user_source_medium_daily` | First-user source / medium |
| `gold_google_analytics_first_user_source_medium_content_daily` | First-user source / medium / content |
| `gold_google_analytics_first_user_manual_source_daily` | Manual first-user source |
| `gold_google_analytics_first_user_manual_source_medium_daily` | Manual first-user source / medium |
| `gold_google_analytics_first_user_manual_source_medium_content_daily` | Manual first-user source / medium / content |
| `gold_google_analytics_property_current` | Current property metadata |
| `gold_google_analytics_event_catalog_current` | Current event catalog |

Dates follow the property timezone.

Each performance table has `*_events_daily` and `*_key_event_pivot_daily`
siblings. Full inventory:

`gold_google_analytics_property_events_daily`
`gold_google_analytics_property_key_event_pivot_daily`
`gold_google_analytics_landing_page_events_daily`
`gold_google_analytics_landing_page_key_event_pivot_daily`
`gold_google_analytics_page_events_daily`
`gold_google_analytics_page_key_event_pivot_daily`
`gold_google_analytics_geography_events_daily`
`gold_google_analytics_geography_key_event_pivot_daily`
`gold_google_analytics_session_channel_events_daily`
`gold_google_analytics_session_channel_key_event_pivot_daily`
`gold_google_analytics_first_user_channel_events_daily`
`gold_google_analytics_first_user_channel_key_event_pivot_daily`
`gold_google_analytics_session_source_events_daily`
`gold_google_analytics_session_source_key_event_pivot_daily`
`gold_google_analytics_session_source_medium_events_daily`
`gold_google_analytics_session_source_medium_key_event_pivot_daily`
`gold_google_analytics_session_source_medium_content_events_daily`
`gold_google_analytics_session_source_medium_content_key_event_pivot_daily`
`gold_google_analytics_session_manual_source_events_daily`
`gold_google_analytics_session_manual_source_key_event_pivot_daily`
`gold_google_analytics_session_manual_source_medium_events_daily`
`gold_google_analytics_session_manual_source_medium_key_event_pivot_daily`
`gold_google_analytics_session_manual_source_medium_content_events_daily`
`gold_google_analytics_session_manual_source_medium_content_key_event_pivot_daily`
`gold_google_analytics_first_user_source_events_daily`
`gold_google_analytics_first_user_source_key_event_pivot_daily`
`gold_google_analytics_first_user_source_medium_events_daily`
`gold_google_analytics_first_user_source_medium_key_event_pivot_daily`
`gold_google_analytics_first_user_source_medium_content_events_daily`
`gold_google_analytics_first_user_source_medium_content_key_event_pivot_daily`
`gold_google_analytics_first_user_manual_source_events_daily`
`gold_google_analytics_first_user_manual_source_key_event_pivot_daily`
`gold_google_analytics_first_user_manual_source_medium_events_daily`
`gold_google_analytics_first_user_manual_source_medium_key_event_pivot_daily`
`gold_google_analytics_first_user_manual_source_medium_content_events_daily`
`gold_google_analytics_first_user_manual_source_medium_content_key_event_pivot_daily`

## Additive / non-additive

`sessions`, `engaged_sessions`, `event_count`, `screen_page_views` are
additive **within one table**. `total_users`, `active_users`, `new_users`
are **non-additive** across dates and dimensions. Daily rows cannot give
deduplicated monthly users. Do not reconstruct property totals by summing
page, geo, or channel breakdowns.

## Ratios

The server lints `SUM`/`AVG` of user counts and stored rates; the finding
is a caveat, not a block.

Recompute engagement rate as `SUM(engaged_sessions)/NULLIF(SUM(sessions),0)`.
Bounces is sessions minus engaged sessions.

## Do not join / do not sum

Do not add manual + resolved families. Do not add session + first-user
families. Do not sum key-event pivot columns as if they were property
totals. Sessions overlap across page paths.

## Attribution

First-user reports group **activity dates** by initial acquisition, not
acquisition cohorts. There is no click/view window column.

## Typical questions → table

| Question | Table |
|---|---|
| Users / sessions this month | `gold_google_analytics_property_daily` (do not SUM users across days as unique) |
| Landing pages | `gold_google_analytics_landing_page_daily` |
| Channel mix | `gold_google_analytics_session_channel_daily` |
