# connector-platform — Claude Code plugin

Add 100+ pre-built integrations (Notion, Slack, Linear, GitHub, …) to your Next.js app with one slash command. Powered by [Connector Platform](https://connector-portal.plataform-connect.workers.dev).

## Install

```
/plugin marketplace add danielsalles/connector-platform-marketplace
/plugin install connector-platform@danielsalles
```

## Quick start

You'll need a builder token (`cpt_*`) from the platform admin.

```
/connector login cpt_xxxxxxxxxxxxxxxxxxxxxxxx
```

This writes `.env.local` and `.mcp.json` at your project root.

```
/connector list
```

Lists available integrations.

```
/connector add notion
```

Scaffolds two Next.js App Router handlers:
- `src/app/api/connect/notion/route.ts` — redirects users to authorize Notion
- `src/app/api/execute/notion/route.ts` — server-side proxy to invoke Notion tools

You can then drop a button in your UI:

```jsx
<a href="/api/connect/notion?user_id={YOUR_USER_ID}">Connect Notion</a>
```

## Commands

| Command | Description |
|---|---|
| `/connector login <token>` | Save credentials and configure MCP |
| `/connector list` | Show available integrations |
| `/connector add <slug>` | Scaffold connect + execute handlers |
| `/connector logout` | Remove credentials and MCP config |

## Frameworks supported

- Next.js (App Router only)

Express / FastAPI / others — open an issue if you need them.

## How it works

The plugin is just markdown — instructions for the Claude Code agent. The actual integration runtime lives in [Connector Platform](https://connector-portal.plataform-connect.workers.dev) (multi-tenant API + MCP server). When you run `/connector add notion`, the agent generates handlers that call the platform's REST API with `fetch`. No SDK — your code stays free of dependencies on this plugin.

## License

MIT — see LICENSE.
