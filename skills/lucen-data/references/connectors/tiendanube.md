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

## The daily date axis is dense

`gold_tiendanube_orders_daily` carries a row for every day between the first
and last order of each `(store_id, currency)` pair, including the days it sold
nothing: every additive column is 0 and `is_zero_filled` is TRUE. A day with no
row is a day outside that window, not a day with no sales.

The window is per currency, not per store. A store that sold in ARS in March
and in USD in April has two windows, and neither covers the other's months, so
on a multi-currency store do not read a missing row as "outside the store's
history" without checking which currency you filtered to.

What this changes for a query:

- `SUM` is unaffected. Adding zeros changes nothing.
- `COUNT(*)` counts calendar days, not days with sales. For the latter,
  filter `NOT is_zero_filled` (or `orders_count > 0`).
- `AVG` over daily rows now includes the zero days. That is the honest daily
  average; say which one you computed when the distinction matters, and
  filter `NOT is_zero_filled` if the question is "on the days it sold".
- Ratios (`aov`, `cancellation_rate`, …) are NULL on a filled day, so they
  drop out of an AVG on their own.
- `is_zero_filled` means "the source reported no orders that day", not "the
  pipeline was healthy". Inside the observed window it cannot tell a real
  zero apart from a missed sync — cross it with source freshness if the
  answer depends on that.

`gold_tiendanube_funnel_daily` and `gold_tiendanube_attribution_daily` are
**not** densified: on those, a day with no row is still a day with no
activity.

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
