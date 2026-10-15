# MCP contracts

## Access and selection

Personal key scopes and stored team/project grants intersect with the owner's current membership and role on each operation. Prefer a read-only, expiring key for discovery. A human edits key access in Personal API Keys; widening grants or extending expiry requires password confirmation. The original key migration assigns only the twelve scopes defined in MCP 0.2.0. Later scopes are opt-in. Existing personal keys migrate to those original scopes and a snapshot of then-accessible projects; future projects are not included.

Scopes: `resources.read`, `diagnostics.read`, `audiences.write`, `topics.write`, `campaigns.draft.write`, `automations.draft.write`, `automation_tests.write`, `automation_runs.manage`, `push_apps.write`, `push.test.send`, `events.ingest`, `automations.trigger`.

Call `account_get`, `teams_list`, `projects_list({ team_id })`, then `project_get({ project_id })`. Account discovery returns scopes/status/version and grant-summary counts; team/project discovery returns the selected IDs and current effective access. Follow discovery pagination, and pass an explicit `project_id` on every project tool.

## Typed inputs

Use the discovered schema and descriptions; unknown fields are rejected. `automations_create/update` and `audience_properties_create/update` retain typed `body` objects. Examples:

```json
{
  "project_id": "PROJECT_UUID",
  "body": { "key": "plan", "type": "string", "sdk_writable": false }
}
```

```json
{
  "project_id": "PROJECT_UUID",
  "body": {
    "name": "Onboarding",
    "definition": {
      "trigger": { "event_name": "customer.created" },
      "steps": [{ "key": "finish", "type": "end_automation", "config": {} }],
      "connections": [{ "from": "trigger", "to": "finish", "branch": "next" }]
    }
  }
}
```

Property keys must exist before contact-field updates. Field values are string, number, boolean, or null. Contact updates/upserts merge supplied fields; null or empty strings remove supplied custom fields.

Use typed targeting definitions with `version: 1`, a channel-appropriate `base`, exclusions, and contact/push rules. Inspect schema/help resources for valid operator/source combinations; do not invent targeting JSON. Definitions are bounded to five group levels and 200 nodes. Read existing content before editing it. Published template versions are distinct from draft versions.

## Automation effects and events

Event ingestion requires `event` plus at least one of `contact_id`, `email`, `external_id`. Use `event_types_list` for names and field types; raw event payloads are not needed for ordinary discovery. Use `automation_builder_options_get` for authoring resources.

Data mutations default to `automation_effects: "suppress"`. For intended live effects, use `automation_effects_preview({ project_id, operation, input })`, inspect affected automations/runs and blockers, then execute with `automation_effects: "trigger"`. `automations.trigger` is required in addition to the operation scope. Suppressed events must not start/resume workflows later.

Use a stable business correlation key/value on trigger and completion events. Validate before sandbox testing; inject synthetic events and advance virtual time for waits. Sandbox events are isolated from live runs.

## Results, pagination, and recovery

Use `response_format: "concise"` for discovery/diagnostics and `"detailed"` for definitions/content. Authoring retrieval tools default to detailed. Read output contracts rather than assuming every resource has the same fields.

For paginated lists, pass `metadata.next` as `after`, preserve filters, and continue while `has_more` is true. Default page limit is bounded; search/filter before fetching large lists. Do not report a first page as the complete population.

Errors retain a safe code, status, field issues, and retry guidance. Correct the indicated paths for validation errors. Explain missing permissions/grants instead of retrying unchanged. Respect retry delay and use the operation's idempotency contract for mutating retries; do not repeat delivery tests blindly after an uncertain response.

## Tests and handoff

`push_test_send` requires app/device plus `campaign_id` or paired title/body. Inspect `push_test_get`, notification trace, and device events; queued or provider-accepted does not mean displayed.

Live Activity start returns `test_id`. Use `push_live_activity_test_get`, update, and end with that handle. Test ownership, project, app, and device remain pinned throughout. End authorized sessions after testing.

`campaigns_preview_audience` and `push_campaigns_preview_audience` provide reach evidence. `dashboard_handoff({ project_id, resource_type, resource_id })` returns a project-aware link for `email_campaign`, `push_campaign`, `automation`, `live_activity_test`, or `import`. Use the returned link; it selects the project only after checking current dashboard access and accepts only allowlisted resource routes.

When messaging tools are enabled and scoped, template draft authoring/restoration, readiness previews, publication, and automation publish/pause/resume/delete use the discovered typed tools and ten-minute prepare/execute receipts. Campaign launch, provider credentials, and domain mutations remain separate rollout workflows. Tool annotations describe side effects; authorization is enforced independently by the server.

## Imports

For staged file or typed-batch imports, read [imports](imports.md). Import merges preserve empty values unless `clear_empty` is explicit, unlike ordinary field updates. New import permissions are never inferred from an existing audience-write scope. Billing workflows remain excluded.

For exports and reviewed data deletion/reallow, read [data operations](data-operations.md). Preserve export cursors and explicit field selection. Never infer the new scopes from an existing read or import permission.
