---
name: lucen-blocks
description: "Author Lucen Blocks dashboards over MCP. Trigger when the user wants to create, edit, restyle, or rearrange a Blocks page/dashboard; change chart colours or types; add KPIs with period comparison; compose rows/stacks/markdown; or anything involving upsert_page / get_page / list_pages for Lucen Blocks."
skill_version: 2026-09-11
# Keep skill_version in sync with `_LUCEN_BLOCKS_SKILL_VERSION` in
# platform/src/lucen_platform/ai/mcp/server.py.
---

# Lucen Blocks authoring guide

This skill **is** the authoring guide. Follow it before `upsert_page` or any restyle. Lucen renders a declarative PageSpec — you never emit React/CSS. Publishing is a human click in the portal; you only write drafts.

Requires the **Lucen MCP server** (write scope) for `upsert_page`. Pair with **lucen-data** for `find_query` / `save_query`. If this skill is not installed, `get_blocks_guide` is an optional MCP fallback — do not call it when this skill is already loaded.

## Before you build: intake

A request like "quiero un tablero de creatividades" names an intent and nothing
else. Do not fill the gaps yourself — the generic answer (KPI strip + bars) is a
different page from the one asked for. Left alone, that is exactly what gets
built.

1. Call `plan_page(request)` with the user's **exact words**. It is deterministic:
   it reads the intent (creatives / funnel / trend / campaigns / overview), lists
   which decisions are open (`open_decisions`: goal, period, metrics, platform,
   most consequential first) and attaches `context` — this organization's relevant
   gold tables with their columns, connected platforms, `data_as_of`, existing pages.
   It does **not** write the questions.
2. If `open_decisions` is non-empty, **you** write the questions: one per open
   decision, at most `max_questions`, in the user's language, all in one message.
   Ground each one in `context` — name the real columns and platforms, offer two or
   three concrete options, mention how fresh the data is when offering periods. A
   question that would fit any organization is a bad question. State the
   `assumptions` you will apply so they can correct them in the same reply, then
   stop. Do not call `find_query`, `save_query` or `upsert_page` until they answer.
   "You decide" is an answer; silence is not.
3. If `existing_page_slug` is set, ask whether to update that page or make a new one.
4. Once `ready_to_build` (or the user answered), restate in one or two lines what
   you will build — widgets, period, metrics — and proceed. Pass the user's words
   to `upsert_page` as `prompt`: the server checks the spec against them and holds
   the write (`state: "needs_confirmation"`) when a creatives / funnel / trend
   request has no `gallery` / `funnel` / `timeseries`. Never pass `confirmed: true`
   unless the user explicitly chose that layout after seeing the mismatch.

Intent words that must change the widget, not just the title: *creativos,
creatividades, anuncios, piezas, imágenes, creatives, ads* → `gallery` (ad-grain
query, `media_id = creative_id`). *Embudo, funnel* → `funnel`. *Evolución,
tendencia, por día, over time* → `timeseries`.

## Authoring loop

0. Intake above. Questions first when the request is one line.
1. `find_query(question)` — HIT → reuse `query_version_id`; MISS → `list_tables` → SQL → `run_bigquery` → user approves → `save_query`
2. **Create:** compose a PageSpec (`title`, `params`, `body`) and `upsert_page(spec, prompt=<user's words>)` → send the user `preview_url`
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
| `emphasis: [{ when, value, means: "good\|bad\|warning\|muted", column? }]` | Threshold colouring (ACOS > 100% → bad). `when` is `above\|at_or_above\|below\|at_or_below\|equals\|not_equals`. Unmatched rows recede to the overflow slate automatically; do not paint them `muted` unless that is the meaning. |
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

## Creative galleries (Meta, MercadoLibre, TiendaNube)

`gallery` draws one card per row: the image, up to 4 footer metrics and a status
chip. `table` can also show images and links per-column (see
[references/widget-catalog.md](references/widget-catalog.md), `## table`) — this
section covers the media-resolution rules both widgets share.
`media_platform` picks how `media_id`
is resolved -- `"meta"` (default), `"mercadolibre"`, `"tiendanube"` -- and each has
its own hard requirement that is easy to miss.

**For `media_platform: "meta"`, the query must be at ad grain.** `media_id` names
the column holding Meta's `creative_id`, and a query aggregated by campaign does not
have one — there is no creative id at campaign grain, so the widget renders nothing.
Group by ad (or by creative), not by campaign or ad set, and select the creative id
column explicitly. `campaign_id`, `adset_id` and `ad_id` are all the wrong column:
only the creative id resolves to an image.

**For `media_platform: "mercadolibre"` / `"tiendanube"`, the query must be at
item/product grain**, and `media_id` must select the gold layer's `thumbnail_url`
column directly — it already holds the final public image URL, not an id.

Meta example:

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

MercadoLibre / TiendaNube example — same shape, one extra field
(`media_platform`), and no `status_column` because these rows carry no ad status:

```json
{
  "type": "gallery",
  "id": "gal_products",
  "title": "Productos",
  "data": { "query": "<query_version_id>" },
  "media_platform": "mercadolibre",
  "media_id": "thumbnail_url",
  "title_column": "item_title",
  "metrics": [
    { "column": "units_sold", "format": "number", "direction": "higher_is_better" },
    { "column": "revenue", "format": "currency_compact", "direction": "higher_is_better" }
  ],
  "sort": { "column": "revenue", "order": "desc" },
  "limit": 24
}
```

`"tiendanube"` is identical except `media_platform: "tiendanube"` and a query
against `gold_tiendanube_product_performance` instead of
`gold_mercadolibre_items`.

- `sort` and `limit` are applied **by the executor** here (and on `bar` / `pie` /
  `table` via `widgets[<id>].order`). Do not also rank and cap in SQL. `limit` defaults to 24
  and is hard-capped at 60; it always bounds the cards drawn. For `media_platform:
  "meta"` it also bounds how many creative ids are resolved and cached per render.
- For **`media_platform: "meta"`**, images are served **by Lucen from its own bucket**,
  never by Meta. Meta's creative URLs are signed and expire within days, so the row
  payload never carries a provider CDN URL — the query supplies the `creative_id` in
  `media_id`. For **`"mercadolibre"` / `"tiendanube"`**, put the gold layer's final
  `thumbnail_url` in `media_id`; Lucen never re-hosts those images.
- For **`"meta"`** only: cards are `pending` on the first render of a new gallery and
  fill in over a short polling cycle as the cache warms. The other states are `ok`,
  `unsupported` (no fetchable still) and `missing` (fetch failed). Grey cards on first
  load are expected, not a bug. **`"mercadolibre"` / `"tiendanube"`** cards are `ok`
  on the first render — no polling.
- A live Meta connection is required only when `media_platform` is `"meta"`.
  `"mercadolibre"` and `"tiendanube"` need no connection because `media_id` already
  holds the image URL.
- For **`"mercadolibre"` / `"tiendanube"`**, `media_id` is validated, not trusted as-is:
  it must be an `https` URL on `*.mlstatic.com` / `*.mitiendanube.com`. Anything else
  (wrong scheme, wrong host, a malformed value) resolves to `missing` instead of
  rendering — a query that selects the wrong column into `media_id` fails visibly as a
  grey/missing card, not by putting an arbitrary string into the page's `<img src>`.
  See [references/widget-catalog.md](references/widget-catalog.md)
  (`## gallery`).

For a `table`, set `type: "image"` on a `ColumnSpec` instead of using a
dedicated media widget — the column's own value is the reference, and
`media_platform` goes on the `table` widget itself (same three values,
same resolution rules as above). See
[references/widget-catalog.md](references/widget-catalog.md) (`## table`)
for the exact shape.

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
