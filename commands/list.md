---
description: List integrations available in Connector Platform (Notion, Slack, etc.)
---

The user wants to see which integrations are available to scaffold.

## Step 1 — Read credentials

Read `.env.local` from project root. Need `CONNECTOR_API_KEY` and `CONNECTOR_BASE_URL`. If missing, reply:

> Not logged in. Run `/connector login <cpt_token>` first.

Stop.

## Step 2 — Fetch the catalog

`GET ${CONNECTOR_BASE_URL}/v1/connectors` with `Authorization: Bearer ${CONNECTOR_API_KEY}`. Use Bash tool with `curl -sS`.

The response is `{ data: Connector[], total }` where each `Connector` has `slug`, `namespace`, `description`, `category`, `iconUrl`.

## Step 3 — Display

Render as a markdown table grouped by `category`:

```
| Slug | Description |
|------|-------------|
| github | GitHub REST API — repos, issues, PRs |
| notion | Notion API — pages, databases |
| ...   | ...         |
```

At the bottom, hint:

> Run `/connector add <slug>` to scaffold a connection (e.g. `/connector add notion`).

If the catalog is empty, say so.
