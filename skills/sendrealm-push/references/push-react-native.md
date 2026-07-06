# React Native Push

Use this reference for React Native Expo and bare React Native apps.

Source docs:

- React Native Expo SDK: `/sdks/react-native-expo`
- React Native Bare SDK: `/sdks/react-native-bare`
- Mobile push credentials: `/tutorials/mobile-push-credentials`
- Send Push Notification API: `/api-reference/endpoint/send-push-notification`

## Contents

- Choose The Path
- Expo Setup Order
- Bare Setup Order
- Initialize
- Permission, Login, Tags, Events
- Notification Opens
- Android Channels
- Diagnostics
- iOS Bare AppDelegate Reminder

## Choose The Path

- Expo Go is not supported because Sendrealm uses native Android and iOS code.
- Use Expo development builds, `expo run`, EAS builds, or prebuild for Expo projects.
- Use the bare flow when the project owns `android` and `ios` directories directly.

## Expo Setup Order

1. Upload Firebase and APNs credentials in Sendrealm.
2. Install `@sendrealm/react-native`.
3. Add the Sendrealm Expo plugin to `app.json` or `app.config.js`.
4. Add Android `google-services.json`.
5. Configure iOS push capabilities.
6. Run Expo prebuild or create an EAS build.
7. Initialize Sendrealm from JavaScript.
8. Request permission after an in-app explanation.
9. Call `login` when the user signs in.
10. Send test pushes on Android and iOS.

```bash
npx expo install @sendrealm/react-native
```

```json
{
  "expo": {
    "plugins": [
      [
        "@sendrealm/react-native",
        {
          "android": {
            "googleServicesFile": "./google-services.json",
            "notificationIcon": "ic_stat_sendrealm",
            "notificationColor": "#111827"
          },
          "ios": {
            "apnsEnvironment": "sandbox",
            "enableBackgroundRemoteNotifications": true,
            "notificationServiceExtension": true
          }
        }
      ]
    ],
    "extra": {
      "sendrealmAppId": "YOUR_SENDREALM_APP_ID",
      "sendrealmPushEnvironment": "development",
      "sendrealmApnsEnvironment": "sandbox"
    }
  }
}
```

Do not put Sendrealm API keys, Firebase service account JSON, or APNs `.p8` files in Expo config.

## Bare Setup Order

1. Upload Firebase and APNs credentials in Sendrealm.
2. Install `@sendrealm/react-native`.
3. Add Android `google-services.json`.
4. Enable iOS Push Notifications capabilities.
5. Forward iOS AppDelegate notification callbacks.
6. Rebuild Android and iOS.
7. Initialize from JavaScript.

```bash
npm install @sendrealm/react-native
npx react-native run-android
npx react-native run-ios
```

If CocoaPods is used directly:

```bash
cd ios
pod install
```

## Initialize

```tsx
import { useEffect } from "react";
import Sendrealm from "@sendrealm/react-native";

export default function App() {
  useEffect(() => {
    void Sendrealm.initialize({
      appId: "YOUR_SENDREALM_APP_ID",
      environment: "development",
      autoRequestPermission: false,
      apnsEnvironment: "sandbox",
    });
  }, []);

  return null;
}
```

Use `environment: "development"` for development devices. Use production settings for TestFlight, App Store, and production Android builds.

## Permission, Login, Tags, Events

```tsx
await Sendrealm.requestPermission();

await Sendrealm.login("user-123", "user@example.com");

await Sendrealm.addTags({
  plan: "pro",
  onboardingComplete: true,
});

await Sendrealm.trackEvent("checkout_started", {
  product_id: "sku_123",
  price: 29,
});
```

Call `login` before setting user tags. Do not use SDK tags for authoritative account, billing, security, compliance, or verified profile data.

## Notification Opens

```tsx
const sub = Sendrealm.addNotificationClickListener((event) => {
  console.log(event.notificationId, event.launchUrl);
});

sub.remove();

const initial = await Sendrealm.getInitialNotification();
```

## Android Channels

Create channels for materially different sound, vibration, or importance.

```tsx
await Sendrealm.createNotificationChannel({
  id: "orders",
  name: "Orders",
  importance: "high",
  soundName: "order_update",
});
```

Android remembers channel behavior. Use a new channel ID when changing behavior users should notice.

## Diagnostics

```tsx
const diagnostics = await Sendrealm.getSupportDiagnostics();
console.log(JSON.stringify(diagnostics, null, 2));
```

Confirm device ID, token presence, permission status, subscribed state, SDK version, and no unexpected SDK error.

## iOS Bare AppDelegate Reminder

Bare apps must forward APNs callbacks from `AppDelegate.swift` and notification responses if the app owns `UNUserNotificationCenter.current().delegate`. Do not skip this when notification opens or iOS token registration fails.
