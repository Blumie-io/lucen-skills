# Layout recipes

Spans are twelfths: `3` = quarter, `4` = third, `6` = half, `12` = full.

## KPI strip (4-up)

```json
{
  "type": "row",
  "id": "kpis",
  "children": [
    { "type": "kpi", "id": "k1", "span": 3, "...": "..." },
    { "type": "kpi", "id": "k2", "span": 3, "...": "..." },
    { "type": "kpi", "id": "k3", "span": 3, "...": "..." },
    { "type": "kpi", "id": "k4", "span": 3, "...": "..." }
  ]
}
```

Cards in the same row stretch to equal height automatically.

## Nested: plot beside prose + two KPIs

```json
{
  "type": "row",
  "id": "main",
  "children": [
    {
      "type": "timeseries",
      "id": "chart",
      "span": 6,
      "chart": "area",
      "height": "lg",
      "data": { "query": "..." },
      "x": "day",
      "y": [{ "column": "revenue", "format": "currency_compact" }]
    },
    {
      "type": "stack",
      "id": "side",
      "span": 6,
      "children": [
        {
          "type": "markdown",
          "id": "note",
          "content": "### Lectura rápida\n\n- Facturación arriba del mes\n- ACOS fuera de rango en un ad group"
        },
        {
          "type": "row",
          "id": "side_kpis",
          "children": [
            {
              "type": "kpi",
              "id": "k_net",
              "span": 6,
              "title": "Neto",
              "data": { "query": "..." },
              "value": {
                "column": "net",
                "format": "currency_compact",
                "direction": "higher_is_better"
              }
            },
            {
              "type": "kpi",
              "id": "k_orders",
              "span": 6,
              "title": "Órdenes",
              "data": { "query": "..." },
              "value": {
                "column": "orders",
                "format": "integer",
                "direction": "higher_is_better"
              }
            }
          ]
        }
      ]
    }
  ]
}
```

## Full-width prose then table then charts

```json
"body": [
  { "type": "markdown", "id": "intro", "content": "## Negocio\n\nResumen…" },
  { "type": "table", "id": "top", "height": "md", "...": "..." },
  { "type": "row", "id": "charts", "children": [ /* bar span 8, pie span 4 */ ] }
]
```

## Emphasis bar (good/bad)

```json
{
  "type": "bar",
  "id": "acos",
  "orientation": "horizontal",
  "data": { "query": "..." },
  "x": "ad_group",
  "y": [{ "column": "acos", "format": "percent" }],
  "emphasis": [
    { "when": "above", "value": 1.0, "means": "bad" },
    { "when": "at_or_below", "value": 1.0, "means": "good" }
  ]
}
```

## Creatives page (Meta)

KPI strip → gallery → supporting table, all on one ad-grain query plus one metrics
query. The `gallery` is full width because the cards already grid themselves (2/3/4
columns by viewport); do not put it in a `span 6` next to a chart.

```json
"body": [
  {
    "type": "row",
    "id": "kpis",
    "children": [
      { "type": "kpi", "id": "k_spend", "span": 4, "title": "Inversión", "data": { "query": "<q_metrics>" }, "value": { "column": "spend", "format": "currency_compact", "direction": "neutral" } },
      { "type": "kpi", "id": "k_roas", "span": 4, "title": "ROAS", "data": { "query": "<q_metrics>" }, "value": { "column": "roas", "format": "ratio", "direction": "higher_is_better" } },
      { "type": "kpi", "id": "k_cpa", "span": 4, "title": "CPA", "data": { "query": "<q_metrics>" }, "value": { "column": "cpa", "format": "currency", "direction": "lower_is_better" } }
    ]
  },
  {
    "type": "gallery",
    "id": "gal_creatives",
    "span": 12,
    "title": "Creativos",
    "subtitle": "Top 24 por inversión en el período",
    "data": { "query": "<q_ads>" },
    "media_id": "creative_id",
    "title_column": "ad_name",
    "status_column": "status",
    "metrics": [
      { "column": "spend", "format": "currency_compact" },
      { "column": "roas", "format": "ratio", "direction": "higher_is_better" },
      { "column": "ctr", "format": "percent" }
    ],
    "sort": { "column": "spend", "order": "desc" },
    "limit": 24
  },
  {
    "type": "table",
    "id": "tbl_ads",
    "height": "lg",
    "title": "Detalle por anuncio",
    "data": { "query": "<q_ads>" },
    "columns": [
      { "column": "ad_name" },
      { "column": "spend", "format": "currency" },
      { "column": "roas", "format": "ratio" },
      { "column": "ctr", "format": "percent" }
    ]
  }
]
```

`<q_ads>` is one query, reused twice: the gallery's `limit` caps the cards, not the
result set, so the table under it still lists every ad the query returned. Because both
widgets read the same `query_version_id` with the same params, the executor runs one
BigQuery job for both.
