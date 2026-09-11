---
name: lucen-blocks
description: "Author Lucen Blocks dashboards over MCP. Trigger when the user wants to create, edit, restyle, or rearrange a Blocks page/dashboard; change chart colours or types; add KPIs with period comparison; compose rows/stacks/markdown; or anything involving upsert_page / get_page / list_pages for Lucen Blocks."
---

# Lucen Blocks authoring guide

This skill **is** the authoring guide. Follow it before `upsert_page` or any restyle. Lucen renders a declarative PageSpec — you never emit React/CSS. Publishing is a human click in the portal; you only write drafts.

Requires the **Lucen MCP server** (write scope) for `upsert_page`. Pair with **lucen-data** for `find_query` / `save_query`. If this skill is not installed, `get_blocks_guide` is an optional MCP fallback — do not call it when this skill is already loaded.

## Authoring loop

1. `find_query(question)` — HIT → reuse `query_version_id`; MISS → `list_tables` → SQL → `run_bigquery` → user approves → `save_query`
2. **Create:** compose a PageSpec (`title`, `params`, `body`) and `upsert_page(spec)` → send the user `preview_url`
3. **Edit:** `get_page(slug)` then `upsert_page(slug, patch=[...])`. Never resend the full body for a small change
4. Iterate on the draft; never claim you published

`upsert_page` takes exactly one of `spec` or `patch`. `patch` requires `slug`. Ops are keyed by node `id`: `set` (title/subtitle/span/height), `remove`, `move`, `insert`, `replace`, `set_page`. A patch copies the rest of the tree, so an unrelated widget cannot disappear.

A later full `spec` write reapplies stored span/height (`layout_prefs`) onto surviving node ids. Prefs for ids that are not in the spec you just wrote are dropped, so a deleted widget cannot haunt a later reincarnation of the same id.

Pass `apply_layout_prefs=false` when the user asked to redo the layout from scratch, **or** when a full spec includes explicit span/height the user just asked for ("make this KPI full width"). Otherwise a previous portal resize silently overwrites the spans you wrote.

To change one widget's width or height, always `patch` a `set`. That updates `layout_prefs`. Do not send a full spec just to change a span.

## Hard rules

- Widgets bind to `query_version_id` via `data.query` — **never inline SQL**
- Layout is a **flow tree**: `row` (horizontal, spans in twelfths) + `stack` (vertical). No x/y/w/h grid
- Every page `params` control must reach every widget (query declares the param) or the widget lists it in `data.ignores`
- Heights are tokens: `sm|md|lg|xl`. Widths are `span` 1–12
- Prefer named palettes; use `#RRGGBB` only when the user asks for a specific colour
- **Do not rewrite the PageSpec to change a span, move a widget, retitle, or delete.** Use `patch`.

## Colour policy

| Prefer | When |
|---|---|
| `palette: "lucen"\|"ocean"\|"mono"\|"org"` on page or widget | Default / restyle |
| `emphasis: [{ when, value, means: "good\|bad\|warning\|muted" }]` | Threshold colouring (ACOS > 100% → bad). Unmatched rows recede to the overflow slate automatically; do not paint them `muted` unless that is the meaning. |
| `y[].color` or `series_colors: { "meta": "#0C2AEA" }` | User named an exact hex |

There is **no** `style` / `css` / `className` field. `"red"` is invalid — use `#DC2626` or `means: "bad"`.

## Params every dashboard should declare

```json
"params": [
  { "type": "date_range", "name": "date_range", "default": "last_30d" },
  { "type": "comparison", "name": "comparison", "default": "previous_period" }
]
```

Add `select` / `multi_select` controls for any dimension the page should slice
(channel, campaign, product). Ship `options` when the values are known and stable;
leave `options` empty only when a dimension cache will fill them. Every widget's
query must declare the same param name, or list it in `data.ignores`.

KPIs with `compare: true` (default) get “↑ 12.6% vs prev 30d” for free — the executor re-runs the query on the shifted window. Set `value.direction` to `higher_is_better`, `lower_is_better`, or `neutral`.

## Chart choice

See [references/chart-decision-tree.md](references/chart-decision-tree.md). Pick the widget from the question (KPI vs trend vs ranking vs composition vs table), not from a gallery.

## Creative galleries (Meta)

`gallery` draws one card per ad creative: the image, up to 4 footer metrics and a
status chip. It is the only widget backed by media, and it has one hard requirement
that is easy to miss.

**The query must be at ad grain.** `media_id` names the column holding Meta's
`creative_id`, and a query aggregated by campaign does not have one — there is no
creative id at campaign grain, so the widget renders nothing. Group by ad (or by
creative), not by campaign or ad set, and select the creative id column explicitly.
`campaign_id`, `adset_id` and `ad_id` are all the wrong column: only the creative id
resolves to an image.

```json
{
  "type": "gallery",
  "id": "gal_creatives",
  "title": "Creativos",
  "data": { "query": "<query_version_id>" },
  "media_id": "creative_id",
  "title_column": "ad_name",
  "status_column": "status",
  "metrics": [
    { "column": "spend", "format": "currency_compact", "direction": "neutral" },
    { "column": "roas", "format": "ratio", "direction": "higher_is_better" }
  ],
  "sort": { "column": "spend", "order": "desc" },
  "limit": 24
}
```

- `sort` and `limit` are applied **by the executor** here, unlike on `bar` / `pie` /
  `table` where they are inert. Do not also rank and cap in SQL. `limit` defaults to 24
  and is hard-capped at 60; it bounds both the cards drawn and the creative images
  resolved per render.
- Images are served **by Lucen from its own bucket**, never by Meta. Meta's creative
  URLs are signed and expire within days, so the row payload never carries a provider
  CDN URL — the only thing the query has to supply is the id.
- Cards are `pending` on the first render of a new gallery and fill in over a short
  polling cycle as the cache warms. The other states are `ok`, `unsupported` (the
  creative has no fetchable still, e.g. a dynamic product ad) and `missing` (the fetch
  was attempted and failed). Grey cards on a first load are expected, not a bug.
- Requires a live Meta connection on the org. `media_platform` is `"meta"`; no other
  platform is supported yet.

## Widget cheat sheet

See [references/widget-catalog.md](references/widget-catalog.md).

## Layout recipes

See [references/layout-recipes.md](references/layout-recipes.md).

## Anti-patterns

See [references/anti-patterns.md](references/anti-patterns.md).

## Markdown

- Page title is H1 — do **not** use `#` for the page name
- `## Section` → Lucen `// SECTION //` chrome
- `###` → normal H3
- GFM + small HTML allowlist (no script, iframe, image, inline style, free class)

## Minimal KPI strip example

```json
{
  "title": "Overview",
  "palette": "lucen",
  "params": [
    { "type": "date_range", "name": "date_range", "default": "last_30d" },
    { "type": "comparison", "name": "comparison", "default": "previous_period" }
  ],
  "body": [
    {
      "type": "row",
      "id": "kpis",
      "children": [
        {
          "type": "kpi",
          "id": "kpi_revenue",
          "span": 3,
          "title": "Facturación",
          "data": { "query": "<query_version_id>" },
          "value": {
            "column": "revenue",
            "format": "currency_compact",
            "direction": "higher_is_better"
          }
        }
      ]
    },
    {
      "type": "timeseries",
      "id": "rev_daily",
      "title": "Evolución",
      "height": "md",
      "chart": "area",
      "data": { "query": "<query_version_id>" },
      "x": "day",
      "y": [{ "column": "revenue", "format": "currency_compact", "label": "Facturación" }]
    }
  ]
}
```
