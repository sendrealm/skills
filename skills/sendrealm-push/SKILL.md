---
name: sendrealm-push
description: Use when building, reviewing, or debugging Sendrealm push notification integrations across backend sends, React Web Push, React Native Expo or bare apps, native Android, native iOS, device registration, provider credentials, permission prompts, login linking, tags, events, diagnostics, direct or audience targeting, and push delivery troubleshooting. Always use this skill when the user asks for Sendrealm push setup, notification sending, or push delivery debugging.
---

# Sendrealm Push

Use this skill for the full push path: configure devices to receive notifications, send notifications from trusted backend code, and debug delivery. Push is best effort; Sendrealm can accept and hand off a send, but device display depends on provider, platform, network, app state, and user settings.

## Choose The Workstream

| User request | Load |
| --- | --- |
| Send a push from server, worker, API route, or queue | [push-send.md](references/push-send.md) |
| Add browser Web Push to React, Vite, or Next.js | [push-web.md](references/push-web.md) and then [push-send.md](references/push-send.md) |
| Add push to React Native Expo or bare RN | [push-react-native.md](references/push-react-native.md) and then [push-send.md](references/push-send.md) |
| Add push to native Android or native iOS | [push-native.md](references/push-native.md) and then [push-send.md](references/push-send.md) |
| Debug missing, delayed, or misrouted notifications | [troubleshooting.md](references/troubleshooting.md) |

## Backend Send Quick Start

```ts
import Sendrealm from "@sendrealm/sdk";

const client = new Sendrealm({
  apiKey: process.env.SENDREALM_API_KEY,
});

await client.push.notifications.send({
  app_id: "push_app_short_id",
  external_ids: ["user_123"],
  notification: {
    title: "Your order shipped",
    body: "Track it now",
    launch_url: "https://example.com/orders/123",
  },
});
```

## Non-Negotiables

- Never expose `SENDREALM_API_KEY`, Firebase service account JSON, or APNs `.p8` keys in browser/mobile code, app config shipped to clients, or public repos.
- Use public Sendrealm Push App IDs in device SDKs; use the API key only in trusted backend code.
- Use exactly one targeting style per send: `tokens`, `web_subscriptions`, `device_ids`, `contact_ids`, `external_ids`, `emails`, or `audiences`.
- Distinguish Sendrealm `environment: "development"` vs `"production"` from iOS `apns_environment: "sandbox"` vs `"production"`.
- Ask permission from a user action; do not auto-prompt unless the product intentionally accepts that UX.
- Call `login` after sign-in before relying on contact, external ID, email, tag, or audience targeting.
- Do not promise guaranteed device display. Provider accepted, queued, or sent counts are not the same as visible notifications.

## Device Setup Order

1. Create or select a Sendrealm Push App and note the public app ID.
2. Configure provider credentials in Sendrealm: Firebase service account for Android, APNs key for iOS, Web provider for browsers.
3. Install the correct device SDK and initialize it once at app startup.
4. Request notification permission from a user gesture.
5. Link the signed-in user with `login(userId, email)` and set app-observed tags or events only after initialization.
6. Send a test push from trusted backend code to a real registered device.
7. Check SDK diagnostics before changing backend targeting logic.
