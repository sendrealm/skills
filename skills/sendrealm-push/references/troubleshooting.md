# Push Troubleshooting

Use this reference when sends are accepted but devices do not show notifications, tokens are missing, or opens route incorrectly.

Source docs:

- SDK overview: `/sdks/overview`
- React Web Push SDK: `/sdks/react`
- React Native Expo SDK: `/sdks/react-native-expo`
- React Native Bare SDK: `/sdks/react-native-bare`
- Android SDK: `/sdks/android`
- iOS SDK: `/sdks/ios`
- Mobile push credentials: `/tutorials/mobile-push-credentials`
- Send Push Notification API: `/api-reference/endpoint/send-push-notification`

## First Triage

1. Confirm the app uses a device SDK and has initialized successfully.
2. Confirm the user granted notification permission.
3. Confirm diagnostics show a Sendrealm device ID and provider token.
4. Confirm `login(userId, email)` ran for identity-based targeting.
5. Confirm the backend send uses the same Sendrealm Push App ID and environment as the registered device.
6. Confirm platform provider credentials are uploaded in Sendrealm and match the app bundle/package.
7. Confirm the response shows expected `sent`, `failed`, `queued`, and notification result values.

## Environment Mismatches

| Symptom | Likely cause |
| --- | --- |
| Development device does not receive production send | Device registered with `environment: "development"` but send omitted development environment. |
| Xcode-installed iOS app does not receive push | APNs environment should usually be `sandbox`. |
| TestFlight/App Store iOS app does not receive push | APNs environment should be `production`. |
| Android token missing | Firebase package name, `google-services.json`, or Google Play services mismatch. |
| Web subscription missing | Service worker path, HTTPS, browser support, permission, or same-origin issue. |

## Diagnostics To Collect

Ask the app to log support-safe diagnostics:

- Sendrealm device ID.
- SDK version.
- Permission status.
- Subscribed state.
- Provider token presence, not necessarily the full token.
- Push environment.
- APNs environment on iOS.
- Last SDK error.
- Service worker check for Web Push.

Do not ask users to paste private provider keys, API keys, `.p8` files, or full service account JSON into chat.

## Platform Checks

### Android

- Device or emulator has Google Play services.
- Firebase Android package name matches `applicationId`.
- `google-services.json` is in the app module.
- Firebase service account JSON is uploaded in Sendrealm, not committed into the app.
- Android notification channel exists and has user-visible importance.
- Status bar icon resource exists and is valid.
- Battery optimization and OEM power settings are not blocking delivery.

### iOS

- Test on a physical device, not only simulator.
- Push Notifications capability is enabled in Apple Developer and Xcode.
- Bundle ID, Team ID, Key ID, APNs `.p8`, and environment match Sendrealm settings.
- Provisioning profile was refreshed after enabling push.
- Notification Service Extension is included for rich images.
- Background Modes with Remote notifications is enabled only when silent/background pushes are required.
- Focus, notification summary, and user notification settings can delay or hide display.

### Web

- The page is served over HTTPS in production.
- The Sendrealm service worker file exists at the configured same-origin path.
- The service worker response is JavaScript, not an app HTML fallback.
- The permission prompt was triggered from a user gesture.
- iOS Web Push requires an installed Home Screen app and manifest support.

## Response Interpretation

- `queued` or `broadcast_id` means an audience/broadcast-style send was queued.
- `sent` means Sendrealm/provider handoff succeeded for those results.
- `failed` means inspect notification result status and provider diagnostics.
- Provider accepted does not guarantee visible display on the physical device.

## Common Fixes

- Reinitialize with the correct `environment`.
- Rebuild native apps after changing Expo plugin, entitlements, push capabilities, service extensions, or Firebase files.
- Rerun `npx @sendrealm/react setup` after updating the React Web Push SDK.
- Use a new Android channel ID when changing sound or importance.
- Call `login` before sending by `external_ids`, `emails`, contact membership, or tags.
- Send to `device_ids` for the narrowest end-to-end test, then widen to users or audiences.
