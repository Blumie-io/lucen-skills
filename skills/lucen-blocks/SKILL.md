---
name: lucen-blocks
description: "Author Lucen Blocks dashboards over MCP. Trigger when the user wants to create, edit, restyle, or rearrange a Blocks page/dashboard; change chart colours or types; add KPIs with period comparison; compose rows/stacks/markdown; or anything involving upsert_page / get_page / list_pages for Lucen Blocks."
---

# Lucen Blocks authoring guide

This skill **is** the authoring guide. Follow it before `upsert_page` or any restyle. Lucen renders a declarative PageSpec — you never emit React/CSS. Publishing is a human click in the portal; you only write drafts.

Requires the **Lucen MCP server** (write scope) for `upsert_page`. Pair with **lucen-data** for `find_query` / `save_query`. If this skill is not installed, `get_blocks_guide` is an optional MCP fallback — do not call it when this skill is already loaded.

## Authoring loop

1. `find_query(question)` — HIT → reuse `query_version_id`; MISS → `list_tables` → SQL → `run_bigquery` → user approves → `save_query`
2. Compose a PageSpec (`title`, `params`, `body`)
3. `upsert_page(spec, slug?)` → send the user `preview_url`
4. Iterate on the draft; never claim you published

## Hard rules

- Widgets bind to `query_version_id` via `data.query` — **never inline SQL**
- Layout is a **flow tree**: `row` (horizontal, spans in twelfths) + `stack` (vertical). No x/y/w/h grid
- Every page `params` control must reach every widget (query declares the param) or the widget lists it in `data.ignores`
- Heights are tokens: `sm|md|lg|xl`. Widths are `span` 1–12
- Prefer named palettes; use `#RRGGBB` only when the user asks for a specific colour

## Colour policy

| Prefer | When |
|---|---|
| `palette: "lucen"\|"ocean"\|"mono"\|"org"` on page or widget | Default / restyle |
| `emphasis: [{ when, value, means: "good\|bad\|warning\|muted" }]` | Threshold colouring (ACOS > 100% → bad) |
| `y[].color` or `series_colors: { "meta": "#0C2AEA" }` | User named an exact hex |

There is **no** `style` / `css` / `className` field. `"red"` is invalid — use `#DC2626` or `means: "bad"`.

## Params every dashboard should declare

```json
"params": [
  { "type": "date_range", "name": "date_range", "default": "last_30d" },
  { "type": "comparison", "name": "comparison", "default": "previous_period" }
]
```

KPIs with `compare: true` (default) get “↑ 12.6% vs prev 30d” for free — the executor re-runs the query on the shifted window. Set `value.direction` to `higher_is_better`, `lower_is_better`, or `neutral`.

## Chart choice

See [references/chart-decision-tree.md](references/chart-decision-tree.md). Pick the widget from the question (KPI vs trend vs ranking vs composition vs table), not from a gallery.

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
