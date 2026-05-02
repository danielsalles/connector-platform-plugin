---
description: Scaffold connect + execute handlers for an integration in this Next.js project
argument-hint: <integration_slug>
---

The user is running `/connector add <slug>` (e.g. `/connector add notion`).

## Step 1 — Read credentials

Read `.env.local`. Need `CONNECTOR_API_KEY`, `CONNECTOR_BASE_URL`, `CONNECTOR_BUILDER_ID`. If missing, error:

> Not logged in. Run `/connector login <cpt_token>` first.

Stop.

## Step 2 — Validate the integration exists

`GET ${CONNECTOR_BASE_URL}/v1/connectors/io.io-platform/<slug>` with bearer. Use Bash + curl.

If 404, reply:

> Integration `<slug>` not found. Run `/connector list` to see what's available.

Save the connector's `name` (human-readable from response).

## Step 3 — Detect framework

Read `package.json`. Required: `next` in `dependencies`. If absent, reply:

> This plugin currently only scaffolds Next.js (App Router) projects. Open an issue if you need Express, FastAPI, or another framework.

Stop.

Detect App Router vs Pages by checking which exists at the project root:
- If `src/app/` or `app/` directory exists → App Router (target this)
- Else → Pages Router

If Pages Router, reply:

> This plugin only scaffolds App Router. Migrate to App Router or open an issue.

Stop.

Determine the source root: `src/app` if `src/` exists, else `app`.

## Step 4 — Write the connect handler

Create `<src_root>/api/connect/<slug>/route.ts`. If file already exists, ask the user before overwriting.

Use this exact template (substitute `<slug>` and `<name>` from step 2):

```typescript
import { NextResponse } from 'next/server';

// Triggers the connect flow for <name>. Redirects the end_user to the
// authorization page (OAuth or API key form) and returns to the success URL
// after they authorize. The end_user is identified by `external_ref` — replace
// the hardcoded value below with your actual user ID lookup (session, JWT, etc).
export async function GET(req: Request) {
  const url = new URL(req.url);
  const externalRef = url.searchParams.get('user_id') ?? 'demo-user';

  // 1. Ensure the end_user exists in Connector Platform (idempotent).
  const userRes = await fetch(`${process.env.CONNECTOR_BASE_URL}/v1/users`, {
    method: 'POST',
    headers: {
      Authorization: `Bearer ${process.env.CONNECTOR_API_KEY}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ external_ref: externalRef }),
  });
  if (!userRes.ok) {
    return NextResponse.json({ error: 'failed to create end_user', detail: await userRes.text() }, { status: 500 });
  }
  const { id: endUserId } = await userRes.json();

  // 2. Create a connect-session that returns an authorization URL.
  const successUrl = `${url.origin}/integrations/<slug>/connected`;
  const sessRes = await fetch(`${process.env.CONNECTOR_BASE_URL}/v1/connect-sessions`, {
    method: 'POST',
    headers: {
      Authorization: `Bearer ${process.env.CONNECTOR_API_KEY}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      end_user_id: endUserId,
      connector_namespace: 'io.io-platform',
      connector_slug: '<slug>',
      success_redirect_url: successUrl,
    }),
  });
  if (!sessRes.ok) {
    return NextResponse.json({ error: 'failed to create session', detail: await sessRes.text() }, { status: 500 });
  }
  const session = await sessRes.json();

  return NextResponse.redirect(session.session_url ?? session.url);
}
```

## Step 5 — Write the execute handler

Create `<src_root>/api/execute/<slug>/route.ts`. Same overwrite check.

```typescript
import { NextResponse } from 'next/server';

// Invokes a tool of the <name> connector for the current end_user.
// Body: { connection_id: string, tool: string, args: Record<string, unknown>, end_user_id: string }
export async function POST(req: Request) {
  const body = await req.json();
  const { connection_id, tool, args, end_user_id } = body;

  if (!connection_id || !tool || !end_user_id) {
    return NextResponse.json({ error: 'connection_id, tool, end_user_id required' }, { status: 400 });
  }

  const res = await fetch(`${process.env.CONNECTOR_BASE_URL}/v1/execute`, {
    method: 'POST',
    headers: {
      Authorization: `Bearer ${process.env.CONNECTOR_API_KEY}`,
      'Content-Type': 'application/json',
      'X-End-User-Id': end_user_id,
    },
    body: JSON.stringify({ connection_id, tool, args: args ?? {} }),
  });

  return new NextResponse(await res.text(), {
    status: res.status,
    headers: { 'Content-Type': 'application/json' },
  });
}
```

## Step 6 — Confirm to the user

Reply:

```
Scaffolded <name> integration.

Files created:
  <src_root>/api/connect/<slug>/route.ts   — initiates authorization flow
  <src_root>/api/execute/<slug>/route.ts   — invokes tools server-side

Suggested button (paste wherever you want):

  <a href="/api/connect/<slug>?user_id={YOUR_USER_ID}">
    Connect <name>
  </a>

After the user authorizes, they'll land on /integrations/<slug>/connected with
?connection_id=... in the URL. Save that connection_id and use it in /api/execute/<slug>
calls together with the user_id as X-End-User-Id.

To call a tool from your code:
  fetch('/api/execute/<slug>', {
    method: 'POST',
    body: JSON.stringify({
      connection_id: '<saved>',
      tool: '<slug>_list_repos',  // see /connector list-tools <slug>
      args: {},
      end_user_id: '<your user id>',
    }),
  })
```

Don't forget: `external_ref` in the connect handler is hardcoded as `'demo-user'` for the demo. Replace it with your real session user id.
