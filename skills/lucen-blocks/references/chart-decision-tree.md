# Chart decision tree

Choose the widget from the **question**, not from a gallery. Overview first,
then trend, then comparison, then detail (Dublin-style dashboards: headline
KPIs with period arrows, time series under them, tables last).

Walk this tree top to bottom. Stop at the first match. Then see
[widget-catalog.md](widget-catalog.md) for props.

```
What is the user trying to see?
│
├─ One headline number for this period (optionally vs last period)
│    └─ kpi
│         compare: true (default) + page comparison param
│         set value.direction: higher_is_better | lower_is_better | neutral
│
├─ How a metric moved over time
│    └─ timeseries  (x = date/time column)
│         ├─ one metric, emphasis on magnitude → chart: "area"
│         ├─ one or more metrics, overlays / crossings → chart: "line"
│         └─ discrete buckets (weeks as categories, not a continuous axis)
│              → chart: "bar"
│         series: grouping column when the user wants one line/area per
│         channel, campaign, …  (palette on the widget; series_colors only
│         for a named hex)
│
├─ Which categories rank or compare (not time)
│    └─ bar  (x = category)
│         ├─ ranking a list of named items (ad groups, SKUs) → orientation:
│         │    "horizontal" is easier to scan, not required. Do not shorten
│         │    names in SQL; the renderer truncates ticks with a hover tooltip
│         ├─ a second grouping column that repeats per x → series (stacked)
│         └─ parts of a whole, 2–8 slices, all values ≥ 0, looks like a share
│              (column name has pct/percent/share/rate, or values sum ~100 / ~1)
│              → pie  (donut: true if you want a hole)
│              Never pie for ranking, time, or more than 8 categories.
│
├─ Conversion through an ordered pipeline (impressions → clicks → purchases)
│    └─ funnel
│         steps: ColumnSpec[] (min 2) from a **single result row**
│
├─ Which ad creatives, products, or catalog items are working (the user wants to SEE them)
│    └─ gallery
│         `"meta"`: ad grain; media_id = Meta creative_id; title_column = ad name
│         `"mercadolibre"` / `"tiendanube"`: item/product grain; media_id = thumbnail_url; title_column = item name
│         metrics: up to 4; sort + limit ARE applied here (executor)
│
├─ Exact values, many rows (>15), mixed column types, or "show me the data"
│    └─ table
│
└─ Explanation, caveats, or section chrome
     └─ markdown
          ## Section → Lucen // SECTION // chrome
          Page title is the H1 — do not use # in markdown for the page name
```

## Default page shape

When the user asks for "a dashboard" / "an overview" and does not specify
layout, compose this and stop:

1. KPI strip (row of 3–4 `kpi`, `span` 4 or 3)
2. One primary `timeseries` (`chart: "area"` or `"line"`, `height: "md"`)
3. Comparison row: `bar` (span 8) + optional `pie` (span 4) **only** if the
   pie rule above holds
4. `table` last if they will want to scan rows

See [layout-recipes.md](layout-recipes.md) for the JSON.

## Hard stops

| Situation | Do not use | Use instead |
|---|---|---|
| Time on X | `bar` as the first choice | `timeseries` |
| "Show me the creatives" | `table` of ad names | `gallery` (ad-grain for meta; item/product-grain for mercadolibre/tiendanube) |
| Ranking a long list of names | `pie` | `bar` (horizontal is easier to scan; vertical is fine — never truncate names in SQL) |
| >8 categories as a share | `pie` | `bar` (limit 8–15) or `table` |
| vs previous period | extra SQL | page `comparison` + KPI `compare: true` |
| Cost / CPA / ACOS KPI | default `direction` | `lower_is_better` |
| Threshold colouring (ACOS > 100%) | hex `"red"` | `emphasis` + `means: "bad"` |
