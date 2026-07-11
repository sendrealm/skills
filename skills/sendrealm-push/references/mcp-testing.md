# MCP And Public API Push Testing

Use this workflow to prove the integration end to end without launching a campaign or sending to a broad target.

## MCP Workflow

1. Call `project_get` and confirm push capabilities.
2. Call `push_apps_list`. Create an app with `push_apps_create` only when needed.
3. Call `push_providers_list`. Report missing provider configuration and hand credential upload to the dashboard.
4. Integrate the public push app ID in the device SDK and complete init, permission, registration, and login.
5. Call `push_devices_list` with the app, platform, and environment. Choose a device with a provider token present and recent activity.
6. Call `push_test_send` with exactly that app and device. Use title/body for an ad hoc test or `campaign_id` for a saved draft.
7. Poll `push_test_get`. Then use `push_notifications_get` for the provider and client event timeline.

Do not substitute a broad direct-send tool for `push_test_send`. MCP intentionally does not expose one.

## Public API And SDK

```ts
const apps = await sendrealm.push.apps.list();
const devices = await sendrealm.push.devices.list({
  app_id: apps[0].public_id,
  environment: "development",
  subscribed: true,
});

const test = await sendrealm.push.tests.create({
  app_id: apps[0].public_id,
  device_id: devices.data[0].device_id,
  title: "Sendrealm test",
  body: "Push registration is working",
});

const trace = await sendrealm.push.tests.retrieve(test.id);
```

Use `push.campaigns.create`, `update`, and `previewAudience` for draft testing. Pass the draft ID to `push.tests.create` to render and deliver it to one device. Scheduling and campaign launch remain dashboard-only.

## Live Activities

MCP exposes explicit test start, update, and end tools. Start must include one selected device ID. Keep a unique `activity_id` per test and always end successful test sessions.

## Interpretation

- No device: initialization or environment/app ID is wrong.
- Device without registration: permission or provider registration did not complete.
- Provider readiness failure: finish FCM/APNs setup in the dashboard.
- Test queued with no notification record: inspect the worker/queue path.
- Provider send event followed by failure: use the provider code and device diagnostics.
- Provider accepted but no display/client event: inspect foreground handlers, notification channels, OS permission, focus modes, and app state.
