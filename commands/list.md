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

The response is `{ data: Connector[], total }` where each `Connector` has `slug`, `namespace`, `description`, `category`, `iconUrl`, `auth_type` (`"oauth2"` | `"api_key"` | `"personal_token"` | `null`).

## Step 3 — Display

Render as a markdown table grouped by `category`. Include the `auth_type` column so the user knows whether they'll need BYO OAuth before going to production:

```
| Slug | Auth | Description |
|------|------|-------------|
| github | oauth2 | GitHub REST API — repos, issues, PRs |
| notion | oauth2 | Notion API — pages, databases |
| resend | api_key | Email delivery |
| ...    | ...    | ...         |
```

At the bottom, hint:

> Run `/connector add <slug>` to scaffold a connection (e.g. `/connector add notion`).
> Connectors with `auth=oauth2` use our shared OAuth app by default. For production, configure your own at <portal>/builder/oauth-apps/<slug>.

If the catalog is empty, say so.
