# Lucen skills

Official agent skills for Lucen. Two audiences:

- **Your team** (Claude, ChatGPT, Cursor one-click): connect the Lucen MCP
  server from the portal (AI tools) and approve access. Stop there. The
  assistant learns Lucen from `get_server_info`, `get_data_guide`, and
  `get_blocks_guide`. No terminal.
- **Developers** (Claude Code, Cursor with a project): connect MCP, then
  optionally install the skills below so the guides live next to the repo.

This repository is a **public mirror**. The source of truth is
[Blumie-io/lucen-platform](https://github.com/Blumie-io/lucen-platform) under
`platform/skills/`. Do not open PRs here; change the skills in lucen-platform.
CI copies `lucen-data` and `lucen-blocks` on every merge to `main`.

## Install (developers)

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

MCP tools query and draft. Connector-only clients get the same rules from
`get_data_guide` and `get_blocks_guide` on the server.
