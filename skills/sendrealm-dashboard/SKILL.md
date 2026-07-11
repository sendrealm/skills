---
name: sendrealm-dashboard
description: Use when creating or updating Sendrealm dashboard resources through the Sendrealm MCP server, including project context, domains, audiences, contacts, topics, templates, automations, email and push campaign drafts, push apps, device diagnostics, and single-device push tests. Always use this skill when an agent must manage Sendrealm dashboard resources rather than only integrate transactional delivery code.
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

## Push Integration Workflow

1. Get the project and confirm the `push` capabilities.
2. List push apps. Create one only when no suitable app exists, then use its `public_id` in the device SDK.
3. List providers and report missing FCM/APNs configuration. Never request provider secrets in chat or upload them through MCP; hand credential setup to a human in the dashboard.
4. List devices for the app and environment. Select a device with `registration_token_present: true`, an allowed permission state, and recent activity.
5. Send one explicit test with `push_test_send`, then poll `push_test_get` and inspect the resulting notification events.
6. If the provider accepted the send but no client event appears, inspect the device SDK and OS state with the `sendrealm-push` skill.
7. For campaign testing, create or update a draft, preview its reachable audience, and pass its ID to `push_test_send`. The test still targets one selected device.

Live Activity testing follows the same boundary: start with one `device_id`, update the returned `activity_id`, and explicitly end the test.

## Safety Rules

- API keys stay in the MCP client configuration or trusted server environment. Never place them in browser or mobile code.
- Use only domains returned by the domain tools. A campaign sender requires a verified project domain.
- Keep contact fields and event payloads to the minimum data needed for the task.
- Read a resource before changing it when an existing ID is supplied.
- Do not use MCP for broad direct push sends or production campaign delivery. `push_test_send` is the only standard push delivery tool and must target one device chosen from the current project.
- Never treat provider acceptance as proof that a notification was displayed. Inspect the test trace and device events.
- Validate automation definitions before publish. For recovery flows, use a stable business correlation key such as `checkout_id` on both trigger and completion events.
- Run automation drafts through no-send test runs before publishing. Test events and virtual-time advancement are isolated from live runs.
- Cancel and retry live automation runs only when explicitly requested; include a clear audit reason and never imply a started delivery can be unsent.

## Supported Workflows

- Inspect project capabilities, domains, DNS records, and tracking settings.
- Create, list, retrieve, and update campaign drafts.
- Create and inspect push apps, inspect redacted provider readiness and notification channels, and diagnose registered devices.
- Create and edit push campaign drafts, preview reachable devices, run a one-device test, and inspect notification delivery traces.
- Start, update, and end a one-device Live Activity test.
- Create, inspect, and update audiences, audience properties, contacts, topics, templates, and automation drafts.
- Look up or upsert contacts by email, validate automations, inspect runs, and cancel or retry eligible live runs.
- Create sandbox test runs, inject matching test events, and fast-forward waits with virtual time.
- Attach or remove a contact from an audience, update topic subscriptions, and ingest correlated events.
