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
| `icon` | Optional emoji or icon token for the page tree |
| `description` | Optional one-line summary |
| `palette` | Default chart palette (`lucen\|ocean\|mono\|org`, default `lucen`) |
| `params` | Header filter bar. See [Page params](#page-params) |
| `body` | Flow tree of nodes |

`spec_version` is always `1` and authors can omit it.

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
| `y` | list of `ColumnSpec` (usually one) |
| `series` | optional grouping column |
| `chart` | `area` (default single-series), `line`, `bar` |
| `stacked` | default `false` — only meaningful with `series` |
| `legend` | default `false` |
| `palette` | `lucen\|ocean\|mono\|org` (default `lucen`) |
| `series_labels` | map series value → legend label. Only with `series`; rejected without it |
| `series_colors` | map series value → `#RRGGBB` |
| `x_label` / `y_label` | axis title overrides |
| `emphasis` | only with `chart: "bar"` and no `series`. `when` includes `equals` / `not_equals`. Unmatched rows recede to the overflow slate (legend "Rest"), not the series accent. Do not use `means: "muted"` as a catch-all; that slot is a declared meaning. Max 4, first match wins |
| `on_click` | `{ action: "set_param", param, value_from, mode, column? }` |
| `height` | `sm\|md\|lg\|xl` |

## bar

Same props as `timeseries` minus `chart`: `x` (category column), `y` (list of
`ColumnSpec`). Plus `orientation: "vertical"|"horizontal"`
(default `vertical`), `sort` (`SortSpec`) and `limit` (`ge=1`). Horizontal is
easier to scan for a ranked list of names; vertical is fine for a handful of
categories. Do not shorten labels in SQL — the renderer ellipsis-truncates ticks
and shows the full text on hover.

`sort` and `limit` are applied by the executor (same helper as `gallery`): it
emits `widgets[<id>].order` as sorted, capped row indices. Shared result rows
are not mutated, so a `table` on the same query still sees every row. `limit`
is `ge=1`. Also takes `legend`, `palette`, `series`, `stacked`, `series_labels`,
`series_colors`, `x_label`, `y_label`, `emphasis`, `on_click`.

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
`badge_column`, `media_platform`, `on_click`.

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
| `image` | The column's own value is a media reference, resolved the same way as `GalleryWidget.media_id`: a Meta `creative_id` when `media_platform: "meta"`, or a direct public image URL when `media_platform: "mercadolibre"` / `"tiendanube"`. Requires `media_platform` on the widget. `size` (`sm` 24px / `md` 32px / `lg` 48px, default `sm`) controls the thumbnail. Not sortable or filterable. `label: ""` is fine — the column needs no header |
| `link` | Renders the column's own (formatted) value as text, linked to the URL in `href_column` (required; must be a column the query returns). Opens in a new tab. A missing, null, or non-string `href_column` value renders plain text, no link |

Image refs are resolved for the rows the executor already selected
(`widgets[<id>].order`: sorted, then `limit`), not for the page the reader
happens to be on. Omit `limit` and every query row is resolved, up to the
query row cap — so with `media_platform: "meta"` the schema requires
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
| `media_id` | **Required.** For `"meta"`: column holding Meta's `creative_id` (one card per distinct id). For `"mercadolibre"` / `"tiendanube"`: column holding the gold layer's `thumbnail_url` — already a direct, public image URL |
| `title_column` | **Required.** Ad name when `media_platform` is `"meta"`; product or item name for `"mercadolibre"` / `"tiendanube"` |
| `metrics` | Up to 4 `ColumnSpec` for the card footer. Four is the readable maximum on one card |
| `status_column` | Column rendered as the Active/Paused chip. `ACTIVE`/`ENABLED`/`LIVE` render green, anything else grey |
| `sort` | `{ column, order }` (`asc` or `desc`, default `desc`). **Applied by the executor.** Omitted = the order the query returned |
| `limit` | Cards rendered per render. For `"meta"` it also bounds how many creative ids are resolved against the cache. Default `24`, hard cap `60`. **Applied by the executor** |
| `media_platform` | `"meta"` (default), `"mercadolibre"`, or `"tiendanube"` |
| `on_click` | `{ action: "set_param", param, value_from: "column", column, mode }`. A card has no category axis and no series, so `value_from: "category"` / `"series"` resolve to nothing and the click is a silent no-op — always use `"column"` here |

**`media_platform: "meta"`** — Query at **ad grain** so every row carries a creative id.
The org needs a live Meta connection. Images are served by Lucen from its own bucket,
never by Meta's CDN: creative URLs are signed and expire within days, so the row
payload never carries a provider URL. The executor resolves each id against Lucen's
`CreativeAsset` cache and returns a side map keyed by widget id.

**`media_platform: "mercadolibre"` / `"tiendanube"`** — Query at **item or product
grain**. Select the gold layer's `thumbnail_url` into `media_id` (`http2.mlstatic.com`
or `acdn-us.mitiendanube.com`). No Meta connection, no cache, no GCS: the executor
validates the URL (`https` + host on the platform's own CDN) and passes it through
unchanged on the first render if it passes; there is no polling and no retry either
way, so the card is `ok` or `missing` immediately.

Card image states:

| State | Means |
|---|---|
| `ok` | `"meta"`: cached image, rendered. `"mercadolibre"` / `"tiendanube"`: the row's own URL passed the host check and is rendered directly |
| `pending` | `"meta"` only. Not cached yet. The first render of a new gallery schedules the fetch and the portal polls (every 4s, up to 10 times) until the cards fill in. `"mercadolibre"` / `"tiendanube"` never reach this state |
| `unsupported` | `"meta"` only. The creative has no fetchable still (dynamic product ads, template creatives) |
| `missing` | `"meta"`: fetch was attempted and failed (Graph error, download error, too large, bad content type), or `media_id` is not a creative id (a CDN URL or other wrong-column value). `"mercadolibre"` / `"tiendanube"`: `media_id` was not an `https` URL on the platform's own CDN — usually the query selected the wrong column |

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

`default` (a `DatePreset`: `today|yesterday|this_week|this_month|last_7d|last_30d|last_90d|last_12m|last_month|this_year`,
default `last_30d`),
`presets` (the control's list; default is `last_7d`, `last_30d`, `last_90d`,
`last_12m`, `today`, `yesterday`, `this_month`, `last_month`, `this_year` —
`this_week` is a valid preset but is **not** in that default list),
`allow_custom` (default `true`), `max_span_days` (`ge=1`).

`name` is frozen to `"date_range"`. Maps to the SQL `@start_date` / `@end_date` pair,
so the query is matched on its date-range control, not on the param name.

## comparison

`default`: `none|previous_period|previous_year` (default `previous_period`). `name` is frozen to `"comparison"`.

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
