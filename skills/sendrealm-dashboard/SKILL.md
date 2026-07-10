---
name: sendrealm-dashboard
description: Use when creating or updating Sendrealm dashboard resources through the Sendrealm MCP server, including project context, sending-domain inspection, audiences, contacts, topics, templates, automations, event correlation, test runs, and email campaign drafts. Always use this skill when an agent must manage Sendrealm dashboard resources rather than only send transactional email or push notifications.
---

# Sendrealm Dashboard

Use the Sendrealm MCP server for API-key-scoped dashboard workflows. Start by getting the current project, then inspect existing resources before creating a duplicate or altering customer data.

## Safe Campaign Workflow

1. Get the current project and capabilities.
2. List domains and select a verified domain.
3. List or create the target audience and confirm its contacts.
4. Create an email campaign draft.
5. Update the draft with sender, subject, audience IDs, tracking preferences, and content or a published template version.
6. Retrieve the campaign and inspect `readiness` before handing it to a human in the dashboard.

Do not claim that MCP can schedule or send campaigns. It deliberately creates and edits drafts only; final delivery approval happens in the SendRealm dashboard.

## Safety Rules

- API keys stay in the MCP client configuration or trusted server environment. Never place them in browser or mobile code.
- Use only domains returned by the domain tools. A campaign sender requires a verified project domain.
- Keep contact fields and event payloads to the minimum data needed for the task.
- Read a resource before changing it when an existing ID is supplied.
- Do not use MCP to send transactional email or push notifications in this version.
- Validate automation definitions before publish. For recovery flows, use a stable business correlation key such as `checkout_id` on both trigger and completion events.
- Run automation drafts through no-send test runs before publishing. Test events and virtual-time advancement are isolated from live runs.
- Cancel and retry live automation runs only when explicitly requested; include a clear audit reason and never imply a started delivery can be unsent.

## Supported Workflows

- Inspect project capabilities, domains, DNS records, and tracking settings.
- Create, list, retrieve, and update campaign drafts.
- Create, inspect, and update audiences, audience properties, contacts, topics, templates, and automation drafts.
- Look up or upsert contacts by email, validate automations, inspect runs, and cancel or retry eligible live runs.
- Create sandbox test runs, inject matching test events, and fast-forward waits with virtual time.
- Attach or remove a contact from an audience, update topic subscriptions, and ingest correlated events.
