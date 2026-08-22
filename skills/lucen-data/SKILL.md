---
name: lucen-data
description: >-
  Query this organization's analytics data (marketing, ad spend, campaign
  performance) through the Lucen MCP tools. Use whenever the user asks about
  their business data, metrics, dashboards, SQL over their warehouse, saved
  queries, or gold-layer tables. Requires the Lucen MCP server to be connected
  (read scope to query; write scope to save queries or draft pages).
---

# Querying Lucen data

Lucen exposes this organization's analytics warehouse (the gold layer in
BigQuery) through MCP tools. Read tools query data; write tools save validated
queries and draft Blocks pages. All access is scoped to the organization the
user approved during setup. Publishing a page is always a human click in the
Lucen portal; MCP only writes drafts.

Requires the **Lucen MCP server** to be connected. If tools are missing, the
user reconnects from the Lucen portal (AI tools) or with `/mcp` in Claude Code.

For dashboard layout, widget choice, palettes, and PageSpec vocabulary, follow
the **lucen-blocks** skill. If that skill is not installed, `get_blocks_guide`
is an optional MCP fallback. Do not call `get_blocks_guide` when lucen-blocks
is already loaded.

## Workflow

1. **Start with `find_query(question)`.** On a hit, run the returned query with
   `run_query`. On a miss, continue below.
2. **Discover schema with `list_tables`** (faster than INFORMATION_SCHEMA), then
   write SQL and test it with `run_bigquery`.
3. When the user approves the SQL you showed them, **save it with `save_query`**.
4. **Build or update a dashboard with `upsert_page`**, then send the user the
   `preview_url` from the response. They publish in Lucen when ready.

Prefer saved queries over ad-hoc SQL whenever one fits. They are cheaper and
already parameterized.

## Building dashboard pages

Follow **lucen-blocks** for widget types, chart choice, layout, and colours.
Hard rules that belong here because they are data-binding, not visual:

- Every data widget's `data.query` must be a `query_version_id` from
  `save_query`, `find_query`, or `list_queries`. Widgets never carry SQL.
- Page `params` are header controls. Every control must reach every widget: each
  pinned query must declare that param, or the widget must list it in
  `data.ignores`.
- `upsert_page` writes a **draft** only. Pages stay drafts until a person
  publishes them in the Lucen portal.

## Tool details

### find_query / list_queries / save_query

`find_query` and `list_queries` return each query's `sql` and an `approval`
field that says where it came from. Only `approval: "reviewed_in_portal"` means
a person opened the version in Lucen's review screen (which shows the SQL) and
approved it. For every other value, show the user the `sql` before pinning that
query into a page other people read.

### run_query(query_version_id, params)

Param values come from the query's `param_schema` (returned by
`list_queries`). The common controls:

- `date_range`: a preset string — `this_week`, `this_month`,
  `last_30d`, `last_90d` — or a custom range:
  `{"preset": "custom", "start": "2026-01-01", "end": "2026-01-31"}`
- `select` params (e.g. `platform`): pass the scalar value, or omit for
  "all".

Omitted params use their defaults.

### run_bigquery(sql)

Constraints (enforced server-side, don't fight them):

- **Single-statement SELECT/WITH only.** No DML/DDL, no multi-statement, no
  scripting. Anything else is rejected.
- **Results are capped at 1000 rows.** Aggregate in SQL instead of pulling raw
  rows; use LIMIT + ORDER BY for top-N questions.
- **A scan-cost gate rejects expensive queries.** If you hit it, narrow the
  date range, select fewer columns, or filter partitions, then retry.

Prefer `list_tables` over INFORMATION_SCHEMA. Saved-query SQL is in
`list_queries` and `find_query` results.

### list_pages / get_page / read_pipeline_code

`list_pages` and `get_page` read Blocks page specs (layout, widgets, params).
`read_pipeline_code` returns dbt/SQL source for a pipeline asset when you need
to understand how a gold table is built.

## Data modeling notes

- Gold tables are flat (one table per breakdown dimension: campaign, geo,
  device, ...). Prefer `WHERE` filters over UNNEST. There are no nested
  `ARRAY<STRUCT>` columns.
- Non-additive metrics (`reach`, `frequency`) live only at campaign grain;
  never SUM them across rows from finer-grain tables.
- Money columns are in the account currency; check column descriptions before
  mixing platforms.

## When something fails

- `401` / auth errors → the user needs to re-authenticate the Lucen MCP
  server (in Claude Code: `/mcp` → Authenticate).
- "This connection is authorized for read-only access" → the token predates
  write support. The user reconnects the MCP server (`/mcp`, re-authenticate)
  and approves the write scope on the consent screen.
- If re-authentication fails with `invalid_scope`, the user removes and
  re-adds the Lucen MCP server to force a fresh dynamic client registration.
- "No data warehouse is configured" → the org hasn't finished onboarding;
  point the user to the Lucen portal.
- Cost-gate rejections are not errors to escalate. Rewrite a cheaper query.
