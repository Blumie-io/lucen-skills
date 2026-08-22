# Widget catalog

Every data widget needs `data: { query: "<query_version_id>", bind?, ignores? }`.

## kpi

Single number + optional period delta.

| Prop | Notes |
|---|---|
| `value` | `ColumnSpec` (`column`, `format`, `direction`, optional `color`) |
| `compare` | default `true` — needs page `comparison` param |
| `title` | Shown as uppercase label |

Formats: `number|integer|compact|currency|currency_compact|percent|ratio|duration_sec`.  
`percent` is a fraction in `[0,1]` (BigQuery `SAFE_DIVIDE`).

## timeseries

| Prop | Notes |
|---|---|
| `x` | date/time column |
| `y` | list of `ColumnSpec` (usually one) |
| `series` | optional grouping column |
| `chart` | `area` (default single-series), `line`, `bar` |
| `palette` | `lucen\|ocean\|mono\|org` |
| `series_colors` | map series value → `#RRGGBB` |
| `emphasis` | only with `chart: "bar"` and no `series` |
| `on_click` | `{ action: "set_param", param, value_from, mode }` |
| `height` | `sm\|md\|lg\|xl` |

## bar

Same as timeseries without `chart`; plus `orientation: "vertical"|"horizontal"`, `sort`, `limit`. Prefer `horizontal` for long category labels.

## pie

`category`, `value` (ColumnSpec), optional `donut`, `series_colors`, `palette`, `limit`.

## funnel

`steps: ColumnSpec[]` (min 2) from a **single result row**. Renderer derives % of first step and step-to-step conversion.

## table

`columns` (order + format + `agg` when `group_by`), `sort`, `limit`, `group_by`, `badge_column`, `on_click`.

## markdown

`content` — GFM + HTML allowlist. Optional `title` → section chrome.

## divider

Hairline rule. No data.

## row

`children` with `span` summing ≤ 12. Children may be widgets or `stack` — **not** another `row`.

## stack

Vertical list of children. Children may be widgets or `row` — **not** another `stack`. Use for “plot | (prose + KPI row)” compositions.
