# Push Sending

Use this reference for backend push sends through `@sendrealm/sdk`.

Source docs:

- JavaScript SDK: `/sdks/javascript`
- Send Push Notification API: `/api-reference/endpoint/send-push-notification`
- SDK overview: `/sdks/overview`
- Mobile push credentials: `/tutorials/mobile-push-credentials`

## Contents

- Install And Configure
- Targeting Rule
- Direct Send Example
- Web-Only Send
- Audience Or Scheduled Send
- Localized Messages
- Buttons And Platform Overrides
- Environment Fields
- Response Handling

## Install And Configure

```bash
npm install @sendrealm/sdk
```

Use `SENDREALM_API_KEY` only in trusted backend code.

```ts
import Sendrealm from "@sendrealm/sdk";

const client = new Sendrealm({
  apiKey: process.env.SENDREALM_API_KEY,
  maxRetries: 2,
});
```

## Targeting Rule

Use exactly one target style per request:

- `tokens`: raw Android FCM or iOS APNs tokens with platform.
- `web_subscriptions`: raw browser `PushSubscription.toJSON()` objects.
- `device_ids`: Sendrealm push device IDs created by a device SDK.
- `contact_ids`: Sendrealm contacts.
- `external_ids`: app user IDs linked by SDK `login`.
- `emails`: contact email addresses.
- `audiences`: audience IDs for queued or scheduled sends.

Use `excluded_audiences` only with contact, external ID, email, or audience sends. Use `platforms` to restrict delivery to `web`, `android`, or `ios`.

The API requires `app_id`, `notification.title`, `notification.body`, and one target style. Target arrays must not be empty.

## Direct Send Example

```ts
await client.push.notifications.send({
  app_id: "push_app_short_id",
  external_ids: ["user_123"],
  notification: {
    title: "Order update",
    body: "Your order is ready.",
    launch_url: "myapp://orders/123",
    image_url: "https://example.com/order.png",
  },
  data: {
    order_id: "123",
  },
});
```

## Web-Only Send

```ts
await client.push.notifications.send({
  app_id: "push_app_short_id",
  emails: ["user@example.com"],
  platforms: ["web"],
  notification: {
    title: "New message",
    body: "Open the inbox.",
    launch_url: "https://app.example.com/inbox",
  },
});
```

## Audience Or Scheduled Send

```ts
await client.push.notifications.send({
  app_id: "push_app_short_id",
  audiences: ["018f5b8d-2e2d-7a9b-9000-1c24e8a3b2c1"],
  excluded_audiences: ["018f5b91-8f46-7f1d-a111-df39fd4b98a0"],
  scheduled_at: "2026-06-21T20:30:00.000Z",
  notification: {
    title: "Sale starts soon",
    body: "Tap to preview.",
  },
});
```

Audience sends are queued through the broadcast delivery pipeline. Direct device, token, contact, external ID, and email sends are delivered immediately.

## Localized Messages

```ts
await client.push.notifications.send({
  app_id: "push_app_short_id",
  contact_ids: ["018f5b8d-2e2d-7a9b-9000-1c24e8a3b2c1"],
  messages: [
    {
      code: "en-US",
      label: "English",
      title: "Welcome back",
      body: "You have a new update.",
    },
    {
      code: "pt-BR",
      label: "Portuguese",
      title: "Bem-vindo de volta",
      body: "Voce tem uma nova atualizacao.",
    },
  ],
  notification: {
    title: "Welcome back",
    body: "You have a new update.",
  },
});
```

Keep `notification` as the fallback/default message.

## Buttons And Platform Overrides

```ts
await client.push.notifications.send({
  app_id: "push_app_short_id",
  device_ids: ["push_device_id"],
  notification: {
    title: "Order ready",
    body: "Pick it up now.",
  },
  buttons: [
    {
      id: "view_order",
      text: "View order",
      launch_url: "myapp://orders/123",
    },
  ],
  android: {
    channel_id: "orders",
    sound: "default",
    ttl: "3600",
  },
  ios: {
    sound: "default",
    badge: 1,
    mutable_content: true,
  },
});
```

Buttons support up to three actions. Android renders button text directly. iOS uses Sendrealm's action payload for SDK tracking and deep-link handling.

Prefer `text` for button labels. `title` is a deprecated alias in the API schema.

## Environment Fields

- `environment: "development"` targets devices registered by development SDK builds.
- Omit `environment`, or set `"production"`, for production devices.
- `ios.apns_environment: "sandbox"` applies to direct APNs token sends and development APNs credentials.
- `ios.apns_environment: "production"` applies to production APNs token sends.

Do not confuse Sendrealm push environment with APNs environment.

## Response Handling

Inspect response totals and notification results, but do not treat provider acceptance as confirmed display.

```ts
const result = await client.push.notifications.send(payload);

console.log({
  total: result.total,
  sent: result.sent,
  failed: result.failed,
  queued: result.queued,
  broadcastId: result.broadcast_id,
});
```

For failures, log sanitized payload context and Sendrealm IDs. Do not log API keys, raw provider private keys, or sensitive custom data.
