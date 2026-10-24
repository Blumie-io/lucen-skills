# Widget catalog

Every data widget needs `data: { query: "<query_version_id>", bind?, ignores? }`.
Precedence of param values is `page` control < widget `bind` < share `locked_params`.

Every node also takes `id` (unique in the page) and `span` (1–12, default `12`,
only meaningful inside a `row`). Every *widget* (not `divider`, `row`, or `stack`)
additionally takes `title`, `subtitle` (one line carrying the insight, not a
second title) and `height` (`sm|md|lg|xl`, default `md`).

# Page

`PageSpec` fields (the document `upsert_page` writes):

| Field | Notes |
|---|---|
| `title` | Page H1 |
| `icon` | Optional. One emoji (`📊`) or `lucide:<name>` from the list below (`lucide:bar-chart-3`). Never a bare name like `chart-line`: it is rejected. Omit it when unsure |
| `description` | Optional one-line summary |
| `palette` | Default chart palette (`lucen\|ocean\|mono\|org`, default `lucen`) |
| `params` | Header filter bar. See [Page params](#page-params) |
| `body` | Flow tree of nodes |

`spec_version` is always `1` and authors can omit it.

Page icon names (`lucide:<name>`): `bar-chart-3`, `line-chart`, `pie-chart`, `trending-up`, `trending-down`, `activity`, `gauge`, `dollar-sign`, `wallet`, `credit-card`, `coins`, `receipt`, `shopping-cart`, `shopping-bag`, `store`, `package`, `truck`, `tag`, `megaphone`, `target`, `sparkles`, `rocket`, `mail`, `message-square`, `share-2`, `bell`, `users`, `user`, `building-2`, `handshake`, `heart`, `file-text`, `folder`, `bookmark`, `book-open`, `image`, `video`, `camera`, `calendar`, `clock`, `globe`, `monitor`, `smartphone`, `database`, `cloud`, `settings`, `search`, `layout-dashboard`, `map`, `home`, `star`, `flag`, `lightbulb`, `zap`, `award`, `gift`.

### Shared shapes

`ColumnSpec`: `column`, optional `label`, `format` (default `number`), `decimals`
(0–6), `direction` (default `neutral`: `higher_is_better\|lower_is_better\|neutral`),
optional `color` (`#RRGGBB`), optional `agg` (`sum\|avg\|none`). There is no
`weighted` agg: a parent ROAS is `SUM(revenue)/SUM(spend)` from SQL, not an
average of ratios.

`SortSpec`: `{ column, order }` where `order` is `asc` or `desc` (default `desc`).
Used by `bar`, `table`, and `gallery`.

`EmphasisRule`: `{ when, value, means, column? }`. `when` is
`above|at_or_above|below|at_or_below|equals|not_equals`. Ordering operators need
a numeric `value`. `column` defaults to the plotted value column. `means` is
`good|bad|warning|muted`. Max 4 rules; first match wins.

`SetParamAction` (`on_click`): `{ action: "set_param", param, value_from, mode, column? }`.
`value_from` is `category|series|column` (default `category`). `mode` is
`replace|toggle` (default `toggle`). `column` is required when `value_from` is
`column`, and forbidden otherwise. `toggle` is only valid against a `multi_select`.

## kpi

Single number + optional period delta.

| Prop | Notes |
|---|---|
| `value` | `ColumnSpec` (`column`, `label`, `format`, `decimals`, `direction`, optional `color`) |
| `compare` | default `true` — needs page `comparison` param |
| `title` | Shown as uppercase label |

Formats: `number|integer|compact|currency|currency_compact|percent|ratio|duration_sec|date|datetime|relative_time|text`.
`percent` is a fraction in `[0,1]` (BigQuery `SAFE_DIVIDE`). `ratio` is a bare
multiplier and is never scaled (`6.22` → `6.22x`). Date formats respect the org
locale; `text` is for dimensions.

## timeseries

| Prop | Notes |
|---|---|
| `x` | date/time column |
| `y` | list of `ColumnSpec`, one per plotted line/area/bar. Each entry's own label, color and format governs its series, its tooltip row and its legend entry. The shared value axis is formatted from y[0] — give same-unit metrics the same format when combining them on one axis. Mutually exclusive with `series` (validated) and with `emphasis` when more than one entry is given (validated): a chart either groups by a series column or plots several y columns, and emphasis needs exactly one |
| `series` | optional grouping column |
| `chart` | `area` (default single-series), `line`, `bar` |
| `stacked` | default `false` — stacks per-series values with `series`, or stacks the `y` columns on top of each other with more than one `y` entry; a no-op with a single `y` entry and no `series` |
| `legend` | default `false`; defaults to `true` (and `chart` to `"line"`) when more than one `y` is given and you did not set them. Duplicate `y` columns are rejected |
| `palette` | `lucen\|ocean\|mono\|org` (default `lucen`) |
| `series_labels` | map series value → legend label. Only with `series`; rejected without it |
| `series_colors` | map series value → `#RRGGBB` |
| `x_label` / `y_label` | axis title overrides |
| `emphasis` | only with `chart: "bar"` and no `series`. `when` includes `equals` / `not_equals`. Unmatched rows recede to the overflow slate (legend "Rest"), not the series accent. Do not use `means: "muted"` as a catch-all; that slot is a declared meaning. Max 4, first match wins |
| `reference_lines` | list of { value, label?, means? }, max 4. Renders a dashed guide on the value axis (a horizontal line for a vertical chart, a vertical line for a `bar` with orientation: "horizontal"). means picks the same good/bad/warning/muted tone as emphasis; omit for a neutral guide |
| `on_click` | `{ action: "set_param", param, value_from, mode, column? }` |
| `height` | `sm\|md\|lg\|xl` |
| `grain` | `{ default, options }`: time-grain toggle. `default` and every entry of `options` is one of `day`, `week`, `month`, `year`; `options` has 2 to 4 unique entries and includes `default`. Only valid when the widget's query declares a STRING param named `grain` (`params: [{"name": "grain", "type": "STRING", "default": "day"}]` on `save_query`); `upsert_page` rejects it otherwise. The viewer picks the grain with a Day / Week / Month / Year toggle (only your `options`), the executor binds the choice, or `default`, to `@grain`, and the x axis labels itself for that grain. The query must group at that grain, see the SQL below. `bind: {grain: ...}` beats the viewer, but needs a page param named `grain` (bind keys must be declared page params). Shared pages show the `default` and no toggle. Only `timeseries` supports it (not `bar`). Patch with `set` and `fields: {grain: {...}}`, or `fields: {grain: null}` to remove it; a `replace` that omits `grain` removes it |

Grain-aware query (BigQuery). Ratios such as ROAS are `SAFE_DIVIDE` of the sums at the
grouped grain, never an average of daily ratios:

```sql
SELECT
  CASE @grain
    WHEN 'week' THEN DATE_TRUNC(date, ISOWEEK)
    WHEN 'month' THEN DATE_TRUNC(date, MONTH)
    WHEN 'year' THEN DATE_TRUNC(date, YEAR)
    ELSE date
  END AS period,
  SUM(spend) AS spend,
  SAFE_DIVIDE(SUM(revenue), SUM(spend)) AS roas
FROM ...
GROUP BY period
ORDER BY period
```

Use `x: "period"` on the widget.

## bar

Same props as `timeseries` minus `chart`: `x` (category column), `y` (list of
`ColumnSpec`, one per plotted bar — see `## timeseries` for the full multi-`y` rules, which apply here identically). Plus `orientation: "vertical"|"horizontal"`
(default `vertical`), `sort` (`SortSpec`) and `limit` (`ge=1`). Horizontal is
easier to scan for a ranked list of names; vertical is fine for a handful of
categories. Do not shorten labels in SQL — the renderer ellipsis-truncates ticks
and shows the full text on hover.

`sort` and `limit` are applied by the executor (same helper as `gallery`): it
emits `widgets[<id>].order` as sorted, capped row indices. Shared result rows
are not mutated, so a `table` on the same query still sees every row. `limit`
is `ge=1`. Also takes `legend`, `palette`, `series`, `stacked`, `series_labels`,
`series_colors`, `x_label`, `y_label`, `emphasis`, `on_click`, `reference_lines`.
`reference_lines` behaves identically to `## timeseries`'s entry: a dashed guide on
the value axis, or a vertical guide when `orientation: "horizontal"`.

## pie

`category`, `value` (ColumnSpec), optional `donut` (default `false`), `legend`
(default `false`), `limit` (`ge=1`), `palette`, `series_colors` (category value
→ `#RRGGBB`), `on_click`.

`limit` is applied by the executor (`widgets[<id>].order`). A pie has no `sort`,
`x_label`, `y_label`, `series`, `stacked`, `emphasis`, or `series_labels`.

## funnel

`steps: ColumnSpec[]` (min 2) from a **single result row**. Renderer derives % of first step and step-to-step conversion.

## table

`columns` (order + format + `agg` when `group_by`; empty `columns: []` means every
column as returned), `sort` (`SortSpec`), `limit` (`ge=1`), `group_by`,
`badge_column`, `media_platform`, `on_click`, `column_widths`.

`column_widths` is `{column: px}`, integers 60-800, only for columns listed in `columns`
(any name when `columns` is empty). A column it omits sizes itself. People set it by
dragging a header edge in edit mode; leave it out when authoring a page. It is stored
as a layout preference like `span` and `height`, so a later `replace` of the table
keeps it. `set` with `column_widths: {}` clears every width.

When `group_by` is set, `columns` cannot be empty and **every** column except the
group key needs `agg` (`sum|avg|none`). `format` cannot decide this: spend and
CPA are both currency.

Formats: `number|integer|compact|currency|currency_compact|percent|ratio|duration_sec|date|datetime|relative_time|text`.
Omit `format` on DATE/TIMESTAMP/STRING columns: the renderer infers `date`/`datetime`/`text`. Never put `number` on a dimension.

`sort` and `limit` are applied by the executor (`widgets[<id>].order`) and the
renderer still paginates the already-capped list. Prefer SQL `ORDER BY`/`LIMIT`
for large result sets.

A `ColumnSpec` can set `type: "image"` or `type: "link"`. Numeric `format`
values (`currency`, `percent`, …) are rejected; omit `format`, or use
`text` / `date` / `datetime` / `relative_time` for a link label. Default
`format: "number"` is kept for dump/reload and is ignored on these columns.

| `type` | Notes |
|---|---|
| `image` | The column's own value is a media reference, resolved the same way as `GalleryWidget.media_id`: a Meta `creative_id` when `media_platform: "meta"`, a Page `post_id` / reel `video_id` when `"facebook_page"`, an Instagram `media_id` when `"instagram"`, or a direct public image URL when `media_platform: "mercadolibre"` / `"tiendanube"`. Requires `media_platform` on the widget. `size` (`sm` 24px / `md` 32px / `lg` 48px, default `sm`) controls the thumbnail. Not sortable or filterable. `label: ""` is fine — the column needs no header |
| `link` | Renders the column's own (formatted) value as text, linked to the URL in `href_column` (required; must be a column the query returns). Opens in a new tab. A missing, null, or non-string `href_column` value renders plain text, no link |

Image refs are resolved for the rows the executor already selected
(`widgets[<id>].order`: sorted, then `limit`), not for the page the reader
happens to be on. Omit `limit` and every query row is resolved, up to the
query row cap — so with a cached `media_platform` (`"meta"`, `"facebook_page"`,
`"instagram"`) the schema requires
`limit` (60 or less, `gallery`'s cap) whenever a column is `type: "image"`.
`mercadolibre` / `tiendanube` need no cap: their refs are validated, not
fetched or cached.

A row whose image reference is null or empty renders an empty cell, never
a broken-image icon. A `meta` ref that is not a creative id at all (the
usual cause: `media_platform: "meta"` over a column of mercadolibre image
URLs) resolves to the same placeholder — it is never fetched, and never
breaks the rest of the page.

## gallery

One card per item: image, metrics footer, status chip. `media_platform` chooses how
`media_id` is interpreted and how the image is resolved.

| Prop | Notes |
|---|---|
| `media_id` | **Required.** For `"meta"`: column holding Meta's `creative_id` (one card per distinct id). For `"facebook_page"`: `post_id` (posts) or `video_id` (reels). For `"instagram"`: the organic `media_id`. Never the gold `image_url` (it expires). For `"mercadolibre"` / `"tiendanube"`: column holding the gold layer's `thumbnail_url` — already a direct, public image URL |
| `title_column` | **Required.** Ad name when `media_platform` is `"meta"`; post text or caption for `"facebook_page"` / `"instagram"`; product or item name for `"mercadolibre"` / `"tiendanube"` |
| `metrics` | Up to 4 `ColumnSpec` for the card footer. Four is the readable maximum on one card |
| `status_column` | Column rendered as the Active/Paused chip. `ACTIVE`/`ENABLED`/`LIVE` render green, anything else grey |
| `sort` | `{ column, order }` (`asc` or `desc`, default `desc`). **Applied by the executor.** Omitted = the order the query returned |
| `limit` | Cards rendered per render. For the cached platforms (`"meta"`, `"facebook_page"`, `"instagram"`) it also bounds how many ids are resolved against the cache. Default `24`, hard cap `60`. **Applied by the executor** |
| `media_platform` | `"meta"` (default), `"facebook_page"`, `"instagram"`, `"mercadolibre"`, or `"tiendanube"` |
| `on_click` | `{ action: "set_param", param, value_from: "column", column, mode }`. A card has no category axis and no series, so `value_from: "category"` / `"series"` resolve to nothing and the click is a silent no-op — always use `"column"` here |

**`media_platform: "meta"`** — Query at **ad grain** so every row carries a creative id.
The org needs a live Meta connection. Images are served by Lucen from its own bucket,
never by Meta's CDN: creative URLs are signed and expire within days, so the row
payload never carries a provider URL. The executor resolves each id against Lucen's
`CreativeAsset` cache and returns a side map keyed by widget id. A carousel creative
renders all its cards in the same card (arrows and dots); nothing to configure.

**`media_platform: "facebook_page"` / `"instagram"`** — Organic content, cached exactly
like `"meta"`. Facebook Page: `gold_facebook_page_post` with `media_id: "post_id"`, or
`gold_facebook_page_reel` with `media_id: "video_id"`. Instagram: `gold_instagram_media`
with `media_id: "media_id"`. The org needs a live `facebook_page` / `instagram_organic`
connection. Reels and videos show their still thumbnail; an album or Instagram
carousel renders all its cards. A text-only Page post is `unsupported`. Putting an
organic id on `"meta"` (the default) fetches it as an ad creative and fails.

**`media_platform: "mercadolibre"` / `"tiendanube"`** — Query at **item or product
grain**. Select the gold layer's `thumbnail_url` into `media_id` (`http2.mlstatic.com`
or `acdn-us.mitiendanube.com`). No Meta connection, no cache, no GCS: the executor
validates the URL (`https` + host on the platform's own CDN) and passes it through
unchanged on the first render if it passes; there is no polling and no retry either
way, so the card is `ok` or `missing` immediately.

Card image states:

| State | Means |
|---|---|
| `ok` | Cached platforms (`"meta"`, `"facebook_page"`, `"instagram"`): cached image, rendered. `"mercadolibre"` / `"tiendanube"`: the row's own URL passed the host check and is rendered directly |
| `pending` | Cached platforms only. Not cached yet. The first render of a new gallery schedules the fetch and the portal polls (every 4s, up to 10 times) until the cards fill in. `"mercadolibre"` / `"tiendanube"` never reach this state |
| `unsupported` | Cached platforms only. No fetchable still (dynamic product ads, template creatives, a text-only Page post) |
| `missing` | Cached platforms: fetch was attempted and failed (Graph error, download error, too large, bad content type, a reel with no still yet), or `media_id` is not an id of that platform (a CDN URL or other wrong-column value). `"mercadolibre"` / `"tiendanube"`: `media_id` was not an `https` URL on the platform's own CDN — usually the query selected the wrong column |

Rows past `limit` are not drawn as cards at all, but they stay in the shared result
payload — a `table` reading the same `query_version_id` still sees every row.

## markdown

`content` — GFM + HTML allowlist. Optional `title` → section chrome. Also inherits
`subtitle` and `height` from every widget, unused by the markdown renderer.

## divider

Hairline rule. No data. A `_Node`, not a `_Widget`: only `id`, `span`, and `type`.
No `title`, `subtitle`, or `height`.

## row

`children` with `span` summing ≤ 12. Children may be widgets or `stack` — **not** another `row`.

## stack

Vertical list of `children`. Children may be widgets or `row` — **not** another `stack`. Use for “plot | (prose + KPI row)” compositions.

---

# Page params

Page `params` are the header filter bar. Every one of them must reach every widget
(the widget's query declares the param, or the widget lists it in `data.ignores`) or
`upsert_page` refuses the write.

All four take `label` and `visible` (default `true`). `visible: false` keeps the param in the page's
param surface — bindable by a widget, lockable by a share link — without rendering a
header control.

## date_range

`default` (a `DatePreset`: `today|yesterday|this_week|last_week|this_month|last_month|this_quarter|last_quarter|this_year|last_7d|last_14d|last_28d|last_30d|last_90d|last_12m`,
default `last_30d`),
`presets` (the control's list; default, in this order: `today`, `yesterday`,
`last_7d`, `last_14d`, `last_28d`, `last_30d`, `last_90d`, `this_week`, `last_week`,
`this_month`, `last_month`, `this_quarter`, `last_quarter`, `this_year`, `last_12m`).
A page that already stores a `presets` array keeps it; the default only applies when
the field is omitted. `last_week` is the previous Monday-Sunday; `this_quarter` is the
calendar quarter to today; `last_quarter` is the previous full calendar quarter.
Rolling presets (`last_7d` ...) include today, so `last_7d` spans 8 days.
`allow_custom` (default `true`), `max_span_days` (`ge=1`).

`name` is frozen to `"date_range"`. Maps to the SQL `@start_date` / `@end_date` pair,
so the query is matched on its date-range control, not on the param name.

## comparison

`default`: `none|previous_period|previous_calendar_period|previous_year` (default `previous_period`). `name` is frozen to `"comparison"`. `previous_calendar_period` compares to the same day-range in the previous calendar month (month-to-date on the 25th vs the 1st-25th of last month), not to the N days immediately before the window -- that is what `previous_period` does. The day-range is shifted, not clamped: a full February (1-28) compares against 1-28 January, and 29-31 March collapses to the last day of February. The portal chip reads "Previous calendar month".

The executor implements it by re-running the query on the shifted window, so it is the
one param a query does not have to declare. This is what feeds KPI `compare: true`.

## select

Single-value dimension filter. `name`, `dimension` (the canonical dimension whose
distinct values populate the options), `options` (optional static list, max 200; empty
means the header falls back to a dimension-keyed options map at render time), `default`.
Maps to a scalar `@<name>` SQL param. A widget's `on_click` targeting a `select` must
use `mode: "replace"`.

## multi_select

Multi-value dimension filter — the platform chips. `name`, `dimension`,
`options` (optional static list, same fallback as `select`),
`default: []`. Maps to an `ARRAY<STRING>` param; empty means "all". The saved SQL must
be written as:

```sql
WHERE (COALESCE(ARRAY_LENGTH(@platforms), 0) = 0 OR platform IN UNNEST(@platforms))
```

The `COALESCE` is not optional: BigQuery coerces an empty ARRAY parameter to NULL, so a
bare `ARRAY_LENGTH(@platforms) = 0` is NULL rather than TRUE and every widget renders
zero rows in the default "nothing selected" state — which is the state every page loads in.
