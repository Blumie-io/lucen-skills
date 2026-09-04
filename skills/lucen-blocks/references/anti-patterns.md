# Anti-patterns

| Don't | Do instead |
|---|---|
| Put SQL in the page spec | `save_query` → pin `query_version_id` |
| Nest `row` inside `row` | `row` → `stack` → `row` |
| Nest `stack` inside `stack` | Flatten or insert a `row` |
| Emit `color: "red"` or CSS | `#DC2626` or `means: "bad"` |
| Compute “vs previous period” in SQL | Page `comparison` param + KPI `compare: true` |
| Leave `direction` default on cost KPIs | `lower_is_better` for CPA/CPC/ACOS |
| Use `#` markdown for the page title | Page `title` field is the H1 |
| Fake headings with bold paragraphs | Real `##` / `###` |
| Filter 3 of 4 widgets with a chip | Every widget’s query declares the param, or `ignores` |
| `bind` a param the query doesn’t declare | Fix the SQL placeholders first |
| Write `ARRAY_LENGTH(@platforms) = 0` for a `multi_select` | `COALESCE(ARRAY_LENGTH(@platforms), 0) = 0` — an empty ARRAY param arrives as NULL |
| Truncate category names in SQL (`LEFT`, `SUBSTR`, `REGEXP_REPLACE`) so the chart fits | Leave the warehouse text intact. The renderer ellipsis-truncates ticks and shows the full label on hover |
| Force `orientation: "horizontal"` only because labels are long | Horizontal is a ranking/readability choice, not a layout workaround |
| Assume `palette: "categorical"` still exists | Use `lucen` / `ocean` / `mono` / `org` |
| Promise you published the page | Send `preview_url`; human clicks Publish |

## `sort` and `limit` are not the same field twice

`bar`, `pie`, `table` and `gallery` all declare `sort` and/or `limit`, and they do
**not** behave the same way. Getting this backwards fails in both directions, and
neither failure raises an error.

| Widget | Who applies `sort` / `limit` |
|---|---|
| `gallery` | **The executor.** It emits `media[<widget_id>].order` already sorted and already capped, and the renderer draws one card per entry — no sorting, no slicing of its own |
| `bar`, `pie`, `table` | **Nobody, yet.** The fields are declared in the DSL and no layer reads them |

| Don't | Do instead |
|---|---|
| Trust `sort` / `limit` on a `bar`, `pie` or `table` to rank or cap what renders | `ORDER BY` + `LIMIT` in the saved query. The widget fields are inert |
| Rank and `LIMIT` a `gallery`'s query in SQL and leave `limit` off the widget | Set `sort` + `limit` on the widget. `limit` also bounds how many creative images get resolved and cached per render (default 24, hard cap 60) |
| Push a `gallery` past 60 cards | The schema refuses it. Narrow the query or add a filter param |

## Gallery

| Don't | Do instead |
|---|---|
| Feed a `gallery` a campaign-grain query | Query at **ad grain**, so every row carries a creative id |
| Point `media_id` at `campaign_id`, `ad_id` or `adset_id` | It must be the column holding Meta's **creative id** — nothing else resolves to an image |
| Put a Meta CDN image URL in the query and expect it to render | Lucen serves the image from its own bucket; Meta's URLs are signed and expire. Only `media_id` matters |
| Report a gallery as broken because the cards are grey on first load | First render is `status: "pending"`; the portal polls until the cache fills. `unsupported` / `missing` are the terminal states |
| Give a card 6 metrics | 4 is the schema maximum, and the readable maximum on one card |
| `on_click` a gallery with `value_from: "category"` or `"series"` | `value_from: "column"` + `column`. A card has neither axis, so the other two resolve to nothing and the click does nothing at all |
