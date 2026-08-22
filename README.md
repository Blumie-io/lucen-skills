# Lucen skills

Official agent skills for Lucen. Install them next to the Lucen MCP server so
Claude, Cursor, and other assistants know how to query your warehouse and
author Blocks dashboards.

This repository is a **public mirror**. The source of truth is
[Blumie-io/lucen-platform](https://github.com/Blumie-io/lucen-platform) under
`agent/skills/`. Do not open PRs here; change the skills in lucen-platform.
CI copies `lucen-data` and `lucen-blocks` on every merge to `main`.

## Install

```bash
npx skills add Blumie-io/lucen-skills --skill lucen-data
npx skills add Blumie-io/lucen-skills --skill lucen-blocks
```

Browse the catalog at [skills.sh/Blumie-io/lucen-skills](https://skills.sh/Blumie-io/lucen-skills).

## Skills

| Skill | Use when |
|---|---|
| `lucen-data` | Query gold-layer analytics, saved queries, and warehouse SQL through Lucen MCP |
| `lucen-blocks` | Author or restyle Lucen Blocks dashboard pages (`upsert_page`, widgets, layout, colours) |

## Two-step setup

1. Connect the Lucen MCP server from the portal (AI tools) or your editor.
2. Install the skills above.

MCP tools query and draft; the skills teach the assistant the workflow. If the
skills are not installed, `get_blocks_guide` on the MCP server is a fallback
for connector-only clients.
