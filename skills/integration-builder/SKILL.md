---
name: integration-builder
description: |
  Use this skill when the user wants to add, integrate, or connect a third-party
  SaaS to their app — phrases like "add Notion to my app", "integrate Slack",
  "connect Linear", "wire up GitHub", "I need Stripe in this project", or
  similar. This skill routes the request to the Connector Platform plugin's
  `/connector add <slug>` command, which scaffolds the necessary handlers in a
  Next.js App Router project.
---

# integration-builder

The user is trying to add an integration to the project. The Connector Platform
plugin already has the runtime — REST API + MCP server with 100+ pre-built
connectors. Don't write integration code from scratch; route to the plugin.

## Decision tree

1. **Detect the integration name** from the user's message. Common ones:
   - notion, slack, linear, github, gitlab, stripe, resend, hubspot, airtable,
     trello, asana, jira, intercom, zendesk, salesforce, mailchimp, sendgrid,
     dropbox, gmail, google-calendar, google-drive, openai, anthropic, etc.
   - Normalize: lowercase, hyphenate spaces, strip "api" suffix.
     "Google Calendar" → `google-calendar`. "Notion API" → `notion`.

2. **Check if `.env.local` exists with `CONNECTOR_API_KEY`**.
   - If yes → run `/connector add <slug>` directly. Stop.
   - If no → instruct the user:

     > To add `<integration>`, you need to log in to Connector Platform first.
     > 1. Get a builder token (`cpt_*`) from your platform admin
     >    (admin runs `Create builder` at `/admin/builders`).
     > 2. Run: `/connector login <token>`
     > 3. Then run: `/connector add <slug>`
     >
     > Want me to check if there's a token available somewhere in this project?

     Then look for `CONNECTOR_API_KEY` in any `.env*` file. If found in another
     env file, instruct the user to copy it to `.env.local` (the plugin reads
     from `.env.local` only).

3. **If the user mentions an integration the platform might not have**, suggest
   they run `/connector list` first to verify availability.

## Confirmation

Before scaffolding, confirm with the user:

> I'll scaffold `<integration>` integration in this project. This will:
> - Create `src/app/api/connect/<slug>/route.ts` (authorization redirect)
> - Create `src/app/api/execute/<slug>/route.ts` (server-side tool invocation)
>
> Proceed?

Wait for explicit confirmation. If the user says yes, run `/connector add <slug>`.

## What NOT to do

- Don't write Notion/Slack/Linear API client code by hand. The whole point of
  the platform is to skip that. The user is paying with a connector token to
  avoid this.
- Don't install npm packages like `@notionhq/client` or `@slack/web-api`. The
  generated handlers use `fetch` directly against the Connector Platform API.
- Don't create the handlers manually — always use the `/connector add` command
  so the templates stay consistent and updates can be applied later.
- Don't suggest the user search for "Notion Next.js tutorial" online; the
  scaffolded handler is enough.

## Examples

**User says**: "Can you add Notion to my app?"
**You do**:
1. Detect slug: `notion`
2. Check `.env.local` — present.
3. Confirm with user (per above).
4. Run `/connector add notion`.

**User says**: "I need to integrate with Slack and Linear."
**You do**:
1. Detect two slugs: `slack` and `linear`.
2. For each, follow the flow (one at a time, confirming separately).
3. After both, suggest the user adds buttons in their UI (point to the snippets
   the `/connector add` command output).

**User says**: "Wire up Stripe payments."
**You do**:
1. Slug: `stripe`.
2. Note: the platform's Stripe connector handles API calls (charges, customers,
   subscriptions). For Stripe Checkout/Elements (payment UI), the user might
   still need Stripe.js. Mention this caveat in your response.
