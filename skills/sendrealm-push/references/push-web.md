# React Web Push

Use this reference for React, Vite, and Next.js browser push registration.

Source docs:

- React Web Push SDK: `/sdks/react`
- Web Push tutorial: `/tutorials/web-push`
- Send Push Notification API: `/api-reference/endpoint/send-push-notification`

## Contents

- Core Rule
- Requirements
- Install
- Initialize In React Or Vite
- Initialize In Next.js
- Link A User
- Tags And Events
- Notification Opens
- Diagnostics
- Common Failures

## Core Rule

Browser code uses `@sendrealm/react` and a public Sendrealm Push App ID. It must never use `SENDREALM_API_KEY`.

## Requirements

- React 18 or newer.
- Sendrealm Push App with Web provider active.
- HTTPS in production.
- Same-origin service worker at `/sendrealm-service-worker.js` by default.
- Permission prompt triggered by a user action.
- iOS Web Push only works for installed Home Screen web apps with a valid web app manifest.

## Install

```bash
npm install @sendrealm/react
npx @sendrealm/react setup
```

The setup command copies `sendrealm-service-worker.js` into the public root. Rerun it after SDK upgrades.

## Initialize In React Or Vite

```tsx
import { useEffect } from "react";
import { init, useSendrealmSubscription } from "@sendrealm/react";

function NotificationButton() {
  const { subscribed, optIn, optOut } = useSendrealmSubscription();

  return (
    <button onClick={() => void (subscribed ? optOut() : optIn())}>
      {subscribed ? "Disable notifications" : "Enable notifications"}
    </button>
  );
}

export function App() {
  useEffect(() => {
    void init({
      appId: import.meta.env.VITE_SENDREALM_PUSH_APP_ID,
      environment: "production",
      autoRequestPermission: false,
    });
  }, []);

  return <NotificationButton />;
}
```

## Initialize In Next.js

Use a Client Component.

```tsx
"use client";

import { useEffect } from "react";
import { init } from "@sendrealm/react";

export function SendrealmInit() {
  useEffect(() => {
    void init({
      appId: process.env.NEXT_PUBLIC_SENDREALM_PUSH_APP_ID!,
      autoRequestPermission: false,
    });
  }, []);

  return null;
}
```

Render `<SendrealmInit />` once near the app shell.

## Link A User

```ts
import { getSendrealmClient } from "@sendrealm/react";

const sendrealm = getSendrealmClient();

await sendrealm.login("user_123", "user@example.com");
```

Call `logout` when the user signs out.

## Tags And Events

Use tags for browser-observed preferences or state.

```ts
await sendrealm.addTags({
  locale: "en-US",
  onboarding_complete: true,
});

await sendrealm.trackEvent("checkout_started", {
  cart_id: "cart_123",
});
```

Use backend-owned contact fields for authoritative account, billing, compliance, security, and verified profile data.

## Notification Opens

```tsx
import { useEffect } from "react";
import { useSendrealm } from "@sendrealm/react";

export function NotificationEvents() {
  const { client } = useSendrealm();

  useEffect(() => {
    const opened = client.addNotificationClickListener((event) => {
      console.log(event.launchUrl, event.notificationId);
    });

    return () => opened.remove();
  }, [client]);

  return null;
}
```

## Diagnostics

```ts
const diagnostics = await getSendrealmClient().getSupportDiagnostics();
console.log(diagnostics);
```

Check browser support, permission status, subscription state, service worker path, device ID, and latest SDK error.

## Common Failures

| Symptom | Check |
| --- | --- |
| Initialization runs during SSR | Move `init()` to a browser-only effect or Client Component. |
| Permission prompt does not appear | Trigger from a user action and check browser site settings. |
| Subscription fails | Confirm service worker is served from the same origin and not an HTML fallback. |
| Notifications open wrong URL | Check `launch_url` and service worker/browser popup behavior. |
| iOS browser does not subscribe | Install the app to Home Screen and verify manifest support. |
