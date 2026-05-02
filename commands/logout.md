---
description: Remove Connector Platform credentials and MCP config from this project
---

The user wants to log out of Connector Platform from this project.

## Steps

1. Remove the `CONNECTOR_*` keys from `.env.local` (CONNECTOR_API_KEY, CONNECTOR_BASE_URL, CONNECTOR_MCP_URL, CONNECTOR_BUILDER_ID). Leave other keys intact. If `.env.local` becomes empty, delete it.
2. Remove the `connector-platform` entry from `mcpServers` in `.mcp.json`. If `mcpServers` becomes empty, delete `.mcp.json` entirely.
3. Confirm:

```
Logged out from Connector Platform.
- Removed CONNECTOR_* keys from .env.local
- Removed connector-platform from .mcp.json

Run `/connector login <cpt_token>` to re-authenticate.
```

Note: this does NOT revoke the token on the server. To revoke, the platform admin needs to delete the token in the admin builders UI.
