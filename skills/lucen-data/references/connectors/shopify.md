# Shopify

## Tables and grain

| Table | Grain | Use for |
|---|---|---|
| `gold_shopify_sales_events_daily` | day (movement date) | Daily sales that match Shopify Analytics: returns on the day of the refund |
| `gold_shopify_orders_daily` | day (purchase date) | Daily orders and revenue, refunds restated to the order's day; cancelled orders out of revenue |
| `gold_shopify_orders` | order | Order-level fact: totals, refunds, net payment, UTMs, customer |
| `gold_shopify_attribution_daily` | day / utm source, medium, campaign | Revenue by last-visit UTM |
| `gold_shopify_customer_ltv` | customer | Lifetime value and audiences (PII, hashed email/phone, consent) |
| `gold_shopify_order_lifecycle` | order | Hours to paid, fulfilled, delivered, closed, cancelled, refunded |
| `gold_shopify_product_performance` | product | Units, gross, discounts, returns, net sales per product |
| `gold_shopify_coupon_performance` | discount code | Redemptions and revenue of orders that used each code |
| `gold_shopify_funnel_daily` | day | Abandoned checkouts vs completed online-store orders |

## Two date views: pick the right one

- "What does Shopify show for last month?" or anything compared with the
  Shopify admin: `gold_shopify_sales_events_daily`. A refund counts on the day
  it was issued, and a cancelled unpaid order appears as a sale and an equal
  return.
- "How did the orders placed last month end up?" or comparisons with ad spend
  of the same days: `gold_shopify_orders_daily`. Refunds are restated to the
  order's day; cancelled orders are excluded from revenue.

Both give the same total over a whole history, and differ day by day whenever
a refund lands on a later day than its sale. Say which view you used.

Days are the shop's local days (its own timezone), like Shopify's reports.

## Additive / non-additive

Amounts and counts on both daily tables are additive across days. Customer LTV
is per customer: do not SUM it into a store total without saying so. In
`gold_shopify_sales_events_daily` all amounts are positive and
`net_sales_amount = gross_sales_amount - discounts_amount - returns_amount`.

## The daily date axis is dense

`gold_shopify_orders_daily` and `gold_shopify_sales_events_daily` carry a row
for every day between the first and last activity of each
`(shop_id, currency)`, with zeros and `is_zero_filled = TRUE` on days without
orders. `COUNT(*)` counts calendar days; filter `NOT is_zero_filled` for days
with sales. Attribution and funnel tables are not densified.

## Ratios

Recompute ratios from their numerators and denominators. Do not AVG a stored
rate (`aov_amount`, `cancellation_rate`, `discount_rate`, ...).

## Do not join / do not sum

- Never use `current_total_amount` as revenue: Shopify leaves it unchanged
  when a refund is issued as an amount. Use `net_payment_amount` (money kept)
  or `net_sales_amount`.
- Product tables cannot see refunds issued without choosing products; their
  returns can be lower than the order-level `returns_amount`.
- Status columns (`financial_status`, `fulfillment_status`) are the CURRENT
  status, not the status on the order's date.

## Attribution

UTMs come from Shopify's customer journey, last visit before the order. Orders
created from drafts, POS or apps have no visit and land in
`direct / none / unknown`. Journey data is computed by Shopify after the order,
so the newest orders can show no UTM for a few hours. It is not Meta's or
Google's attribution window.

## Typical questions → table

| Question | Table |
|---|---|
| Net sales this month, as Shopify shows it | `gold_shopify_sales_events_daily` |
| Revenue of the orders placed this month | `gold_shopify_orders_daily` |
| Best products | `gold_shopify_product_performance` |
| Revenue by campaign | `gold_shopify_attribution_daily` |
| Customers to target / audience export | `gold_shopify_customer_ltv` |
| Coupon results | `gold_shopify_coupon_performance` |
| Abandonment rate | `gold_shopify_funnel_daily` |
| How fast orders ship | `gold_shopify_order_lifecycle` |
