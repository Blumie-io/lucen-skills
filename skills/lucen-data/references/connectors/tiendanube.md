# Tiendanube

## Tables and grain

| Table | Grain | Use for |
|---|---|---|
| `gold_tiendanube_orders_daily` | day | Daily order / revenue totals |
| `gold_tiendanube_orders` | order | Order-level fact |
| `gold_tiendanube_attribution_daily` | day / source | Last-click style daily attribution |
| `gold_tiendanube_customer_ltv` | customer | Lifetime value. Not summable as unique customers across filters without care |
| `gold_tiendanube_order_lifecycle` | order | Status timeline |
| `gold_tiendanube_product_performance` | product | Product sales |
| `gold_tiendanube_coupon_performance` | coupon | Coupon usage |
| `gold_tiendanube_funnel_daily` | day | Store funnel |

## Additive / non-additive

Daily revenue and order counts on `gold_tiendanube_orders_daily` are additive
across days. Customer LTV is **per customer** — do not SUM LTV to invent a
store total without saying so. Funnel steps are not additive across steps.

## Ratios

The server lints `SUM`/`AVG` of stored rates; the finding is a caveat, not
a block.

Recompute conversion rates from funnel numerators and denominators. Do not
AVG a stored rate.

## Do not join / do not sum

Do not add daily totals to order-level rows. Refunds live on
`gold_tiendanube_orders`; daily totals already have their own convention
(see column descriptions).

## Attribution

`gold_tiendanube_attribution_daily` is the store's attribution grain. It is
not Meta or Google's click window.

## Typical questions → table

| Question | Table |
|---|---|
| Revenue this month | `gold_tiendanube_orders_daily` |
| Best products | `gold_tiendanube_product_performance` |
| Coupon lift | `gold_tiendanube_coupon_performance` |
