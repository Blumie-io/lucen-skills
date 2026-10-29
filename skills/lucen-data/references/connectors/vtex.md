# VTEX

## Tables and grain

| Table | Grain | Use for |
|---|---|---|
| `gold_vtex_orders` | order (`account_name`, `order_id`) | Order-level revenue and status |

## Additive / non-additive

`order_value`, `total_units`, and `items_selling_amount` sum across orders only when every row shares the same `currency_code`. Do not sum orders across accounts without filtering to one `account_name`.

## Ratios

Recompute any rate from summed parts (for example average order value = `SUM(order_value) / COUNT(*)`) instead of averaging stored totals.

## Do not join / do not sum

Do not join `gold_vtex_orders` to itself on different dates expecting one row per order: the table already holds the latest fetch inside the pipeline lookback. Line items live in staging only; gold exposes aggregated item counts and amounts.

## Attribution

VTEX OMS data is store order facts, not ad-platform attribution. UTM or marketplace origin fields on the order (`origin`, `sales_channel`) describe the store's own classification, not Meta or Google click windows.

## Typical questions → table

| Question | Table |
|---|---|
| Order revenue by day | `gold_vtex_orders` (group by `order_created_date`, same currency) |
| Order status mix | `gold_vtex_orders` |
| Units sold | `gold_vtex_orders` (`total_units`) |
