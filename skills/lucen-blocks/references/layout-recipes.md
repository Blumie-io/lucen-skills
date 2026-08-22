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
