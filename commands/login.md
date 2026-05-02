---
description: Save your Connector Platform builder token and configure MCP for this project
argument-hint: <cpt_token>
---

The user is running `/connector login` with a builder token (`cpt_*`) provided by the Connector Platform admin.

## Step 1 — Validate the token format

The token must start with `cpt_`. If the user passed something else (or nothing), reply:

> The token must start with `cpt_`. Get one from your platform admin (admin runs `Create builder` in the platform's `/admin/builders` page and shares the bootstrap token).

Stop here on format error.

## Step 2 — Verify the token works and is a builder token

Make a GET request to `https://connector-api.plataform-connect.workers.dev/v1/me` with header `Authorization: Bearer <token>`. Use the Bash tool with `curl -sS`.

Parse the JSON response:
- If status != 200 → tell the user the token is invalid or revoked. Stop.
- If `role` != `"builder"` → tell the user this is not a builder token (it might be an end_user or admin token). Stop.
- If `builder_id` is null → tell the user the token has no builder context. Stop.

Save `builder_id` from the response — you'll need it next.

## Step 3 — Write `.env.local` at project root

If `.env.local` already exists, **append** these lines (or update if they already exist). Do NOT clobber existing keys.

```
CONNECTOR_API_KEY=<token>
CONNECTOR_BASE_URL=https://connector-api.plataform-connect.workers.dev
CONNECTOR_MCP_URL=https://connector-mcp.plataform-connect.workers.dev
CONNECTOR_BUILDER_ID=<builder_id from step 2>
```

## Step 4 — Write `.mcp.json` at project root

Write the file with exactly this content (substitute `<builder_id>` and `<token>`):

```json
{
  "mcpServers": {
    "connector-platform": {
      "url": "https://connector-mcp.plataform-connect.workers.dev/mcp/<builder_id>/self",
      "headers": {
        "Authorization": "Bearer <token>"
      }
    }
  }
}
```

If `.mcp.json` already exists with other servers, merge — preserve existing keys, add `connector-platform` to `mcpServers`.

## Step 5 — Update `.gitignore`

Ensure `.env.local` is in `.gitignore`. Append the line if missing. Do not touch other entries.

## Step 6 — Confirm to the user

Reply with exactly this format:

```
Logged in to Connector Platform.
- Builder: <builder_id from step 2>
- Wrote .env.local
- Wrote .mcp.json (you may need to restart Claude Code for the MCP server to load)
- Updated .gitignore

Next: run `/connector add <integration>` to scaffold a connection (e.g. `/connector add notion`).
List available integrations with `/connector list`.
```

Important: never log or echo the full token in your output to the user — only the `cpt_xxxxxxxx…` first 12 chars max.
