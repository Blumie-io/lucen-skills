# MercadoLibre

## Tables and grain

| Table | Grain | Use for |
|---|---|---|
| `gold_mercadolibre_ads_campaign_performance` | campaign / day | Product Ads campaigns |
| `gold_mercadolibre_ads_ad_group_performance` | ad group / day | Ad group delivery |
| `gold_mercadolibre_ads_ad_group_items` | ad group / item | Items in an ad group |
| `gold_mercadolibre_sales_daily` | day | Reporting-ready commercial sales |
| `gold_mercadolibre_sales_by_region_daily` | region / day | Sales geo |
| `gold_mercadolibre_orders_status_daily` | status / day | Order counts by status |
| `gold_mercadolibre_orders_status_by_region_daily` | status / region / day | Status geo |
| `gold_mercadolibre_financial_events_daily` | day | Billing / money movements |
| `gold_mercadolibre_cancellations` | cancellation | Cancelled orders |
| `gold_mercadolibre_pack_split_lineage` | pack / order | Split-pack lineage |
| `gold_mercadolibre_items` | item | Item catalog |
| `gold_mercadolibre_order_geography` | order | Order geo grain |

## Additive / non-additive

Sales amounts on `gold_mercadolibre_sales_daily` are the commercial net to
sum. `net_sales_amount` is the Mercado Libre Ventas report Total before taxes
(gross minus sale fee minus shipping cost), so it runs a few percent above the
report's Total (ARS). Say so when comparing to a seller's export. If
`net_input_status` is not `complete`, some orders are missing from the net;
report it as partial. Order **counts** on status tables are not additive across statuses
(the same order can appear in more than one snapshot). Do not add
`gold_mercadolibre_sales_daily` to `gold_mercadolibre_orders_status_daily`.

## Ratios

The server lints `SUM`/`AVG` of ACOS / ROAS; the finding is a caveat, not
a block.

Recompute ACOS / ROAS from SUM(ads spend) and SUM(sales) at a compatible
grain. Do not average those ratios.

## Do not join / do not sum

Do not add financial events to commercial sales to invent "net". Financial
events are billing; sales_daily is commercial net. Ads facts do not replace
sales facts.

## Attribution

Product Ads delivery and MercadoLibre orders are separate. There is no
paid-click attribution window tying an order to an ad in these gold tables.

## Typical questions → table

| Question | Table |
|---|---|
| Daily sales | `gold_mercadolibre_sales_daily` |
| Ads spend | `gold_mercadolibre_ads_campaign_performance` |
| Cancellations | `gold_mercadolibre_cancellations` |
