# MercadoLibre

## Reporting destination

Always qualify the dataset. Regular Mercado Libre reporting uses `{client}_gold`
with the provider-backed contract below. Existing table names remain; their
financial definitions and column set replace the previous contract. Do not use
`gold_v2` as a current source: retained comparison tables are historical and are
not refreshed automatically. Other connectors keep their existing gold routes.

Existing dashboard SQL using retired fields needs review before the first gold
rebuild. Keep net status/cutoff visible, include adjustment-only rows and do not
assume matching table names imply matching financial definitions. Invoice,
perception, discount and supporting shipment-cost diagnostics run only in isolated
engineering schemas; they are not regular gold reporting inputs.

## Tables and grain

| Table | Grain | Use for |
|---|---|---|
| `gold_mercadolibre_ads_campaign_performance` | campaign / day | Product Ads campaigns |
| `gold_mercadolibre_ads_ad_group_performance` | ad group / day | Ad group delivery |
| `gold_mercadolibre_ads_ad_group_items` | ad group / item | Items in an ad group |
| `gold_mercadolibre_sales_daily` | seller / currency / product variation / status / day | Commercial measures and provider net, including net-only adjustments |
| `gold_mercadolibre_sales_by_region_daily` | seller / currency / region / day | Commercial measures and provider net by geography |
| `gold_mercadolibre_orders_status_daily` | status / day | Order counts by status |
| `gold_mercadolibre_orders_status_by_region_daily` | status / region / day | Status geo |
| `gold_mercadolibre_cancellations` | cancellation | Cancelled orders |
| `gold_mercadolibre_pack_split_lineage` | pack / order | Split-pack lineage |
| `gold_mercadolibre_items` | item | Item catalog |
| `gold_mercadolibre_order_geography` | order | Order geo grain |

`gold_mercadolibre_financial_events_daily` is an engineering diagnostic model,
not a regular gold report. It publishes invoice-event audit detail into the
explicit per-run engineering schema; do not use a retained gold copy as a current
sales source or add its amounts to canonical net.

Mercado Pago uses the existing Mercado Libre seller connection. Canonical net is
`mercadolibre_payment_billing`: payment net, normalized for embedded shipping,
confirmed seller-funded reversals and cumulative charge refunds, combined with
actual billed shipping/return charges and linked credits. It is a sale's financial
result, not verified account-balance movements or available cash. Never subtract
payment fees/taxes again or add invoice representations of those deductions.
Unlinked funding, withdrawals, advertising and account activity are outside sales.

## Additive / non-additive

Daily and regional sales read one validated sales-line snapshot. `net_sales_amount`
is payment-and-billing net, with `net_sales_source`, `net_data_status` and `net_data_as_of`.
They are tables: amounts and status remain as last built until the next successful
dbt rebuild. A failure before dbt cannot update their status. Check financial
cutoff and pipeline freshness, not status alone.
Refunds revise the original sale month. Ventas is a comparison with its own
population and cutoff; do not promise exact equality to its Total.

Gross, units and order counts retain the paid/partially-refunded
commercial population. Cancelled or refunded sales can contribute net adjustments
with zero commercial measures. Do not restrict net queries to paid statuses.

Available valid components publish net with partial status; no usable amount
remains NULL. A stale value retains its last validated amount and cutoff. Sum
available net within one currency and carry incomplete/partial status if any
contributing row is partial or unavailable; an all-unknown result remains NULL.
Never coalesce missing net to zero. Expose status and the earliest source cutoff
beside totals. Partial net is an observed subtotal, not a complete settlement.

Sales gold exposes `sale_fee_amount` (actual seller fee-type payment charges after returned
fees), `tax_deduction_amount` (seller payment taxes after returned taxes),
`shipping_cost_amount` (billed shipping/returns after linked credits),
`refund_amount` (seller-funded principal reversals, excluding verified protected
refunds) and `coupon_fee_amount` (seller coupon charges after credits). Fee/tax
amounts include linked shipment payments. Buyer charges, marketplace subsidies
and unlinked account expenses are excluded. These are already included in net;
never subtract them again. The fee column is not the legacy order commission estimate.

Breakdowns use the same sale/product attribution and exact-cent allocation policy
as net, but each amount is allocated independently. They are not a guaranteed
gross-minus-costs bridge at product grain. Include adjustment-only statuses.
Use the same NULL-preserving aggregation for each amount. Missing evidence or an
amount that cannot be represented in currency minor units remains NULL; net's
status alone does not guarantee all breakdowns are known. Failed refreshes retain
the previous breakdowns with net and its cutoff; an older publication without the
new breakdowns leaves those NULL until a complete refresh.

Product-level distinct order counts are not additive across products. Order
counts on status tables are not additive across status snapshots. Do not add
daily, regional and order-status facts together; choose the appropriate grain.

## Ratios

The server lints `SUM`/`AVG` of ACOS / ROAS; the finding is a caveat, not
a block.

Recompute ACOS / ROAS from SUM(ads spend) and SUM(sales) at a compatible
grain. Do not average those ratios.

## Do not join / do not sum

Do not add invoice financial events, payment snapshots or audit allocations to
canonical sales net. They are overlapping audit surfaces at different grains.
Sales gold omits legacy competing net variants, billing/tax estimates and
payment/shipping diagnostics. Its five explanatory cost/refund columns use the
provider definitions above. Detailed evidence belongs in engineering audit facts.
Pre-aggregate audit facts before joining sales; their charges must
not be deducted from provider net again. Ads facts do not replace sales facts.

## Attribution

Product Ads delivery and MercadoLibre orders are separate. There is no
paid-click attribution window tying an order to an ad in these gold tables.

## Typical questions → table

| Question | Table |
|---|---|
| Daily sales | `gold_mercadolibre_sales_daily` |
| Ads spend | `gold_mercadolibre_ads_campaign_performance` |
| Cancellations | `gold_mercadolibre_cancellations` |
