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
| Vertical bars with 20-char labels | `orientation: "horizontal"` |
| Assume `palette: "categorical"` still exists | Use `lucen` / `ocean` / `mono` / `org` |
| Promise you published the page | Send `preview_url`; human clicks Publish |
