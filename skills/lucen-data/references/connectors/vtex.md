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

VTEX OMS data is store order facts, not ad-platform attribution. `origin` and `sales_channel` are the store's own classification, not Meta or Google click windows.

`utm_source`, `utm_medium` and `utm_campaign` are the last-touch UTMs the store recorded on each order (VTEX `marketingData`). Blank values are NULL. Use them to split store sales by platform and by paid vs organic; they will not match the conversions an ad platform reports, which use its own click and view windows.

Platform from `utm_source` (case-insensitive):

| `utm_source` | Platform |
|---|---|
| `google` | Google |
| `fb`, `ig`, `meta`, `facebook`, `instagram` | Meta |
| `tiktok` | TikTok |
| NULL | No UTM (direct, organic search or untagged) |

Paid vs organic. Compare `utm_source` and `utm_medium` case-insensitively (`LOWER(...)`). Apply the rules in this order; the first match wins:

1. Organic social: a Meta source and `utm_medium` is `social` and `utm_campaign IS NULL`.
2. Paid: `utm_medium` is `cpc` or `paid`, or contains `pauta` (a store's own paid tag, for example `pautacybermonday26`), or a Meta placement (`instagram_feed`, `instagram_stories`, `facebook_mobile_feed`, ...), or a Meta source with `utm_campaign IS NOT NULL`.
3. Anything else: report the raw `utm_source` / `utm_medium` / `utm_campaign` and do not invent a channel.

`utm_campaign` is whatever the store recorded: a Meta campaign id, a Google campaign name or free text. Do not assume it joins to ad-platform campaign ids.

Exclude cancelled orders (`status = 'canceled'`) when counting sales, as for any store revenue.

## Freshness

Gold is refreshed daily; the portal order views are real time.

## Typical questions → table

| Question | Table |
|---|---|
| Order revenue by day | `gold_vtex_orders` (group by `order_created_date`, same currency) |
| Order status mix | `gold_vtex_orders` |
| Units sold | `gold_vtex_orders` (`total_units`) |
| Store sales by platform (UTM) | `gold_vtex_orders` (group by mapped `utm_source`) |
| Paid vs organic store sales | `gold_vtex_orders` (`utm_medium`, `utm_campaign`, rules above) |
