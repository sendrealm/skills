# Native Android And iOS Push

Use this reference for native Android Kotlin/Java and native iOS Swift/Objective-C apps.

Source docs:

- Android SDK: `/sdks/android`
- iOS SDK: `/sdks/ios`
- Mobile push credentials: `/tutorials/mobile-push-credentials`
- Send Push Notification API: `/api-reference/endpoint/send-push-notification`

## Contents

- Android Requirements
- Android Setup
- Android Initialize
- iOS Requirements
- iOS Install
- iOS Initialize
- Identity, Tags, Events
- Rich And Background iOS Push
- Diagnostics

## Android Requirements

- Android API 27 or newer.
- Google Play services on target devices or emulators.
- Sendrealm Push App ID.
- Firebase Cloud Messaging configured for the same Android package name.
- `google-services.json` added to the Android app.
- Firebase service account JSON uploaded in Sendrealm.

## Android Setup

Add the Sendrealm Android dependency using the current published version from the docs or Maven Central.

```kotlin
dependencies {
    implementation("com.sendrealm:sendrealm-android:0.1.2")
}
```

Apply Google Services if the app does not already use it.

```kotlin
plugins {
    id("com.google.gms.google-services")
}
```

Place `google-services.json` at:

```text
app/google-services.json
```

The Firebase Android package name must match the app `applicationId`.

## Android Initialize

```kotlin
import android.os.Bundle
import androidx.activity.ComponentActivity
import com.sendrealm.sdk.Sendrealm
import com.sendrealm.sdk.SendrealmConfig

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val config = SendrealmConfig()
            .setEnvironment("development")
            .setAutoRequestPermission(false)

        Sendrealm.initialize(
            context = this,
            appId = "YOUR_SENDREALM_APP_ID",
            config = config
        )
    }
}
```

Ask permission from a user action:

```kotlin
Sendrealm.requestPermission(this)
```

Forward new launch intents so notification taps can be tracked and routed:

```kotlin
override fun onNewIntent(intent: Intent) {
    super.onNewIntent(intent)
    setIntent(intent)
    Sendrealm.handleNotificationOpen(intent)
}
```

## iOS Requirements

- iOS 13.4 or newer.
- Sendrealm Push App ID.
- Push Notifications enabled for the Apple App ID and Xcode target.
- APNs `.p8` key, Key ID, Team ID, Bundle ID, and environment uploaded in Sendrealm.
- Physical iOS device for end-to-end push testing.

## iOS Install

Swift Package Manager:

```text
https://github.com/sendrealm/ios.git
```

CocoaPods:

```ruby
pod "SendrealmIOS"
```

## iOS Initialize

```swift
import SendrealmIOS

Sendrealm.configure()

Sendrealm.shared.initialize([
  "appId": "YOUR_SENDREALM_APP_ID",
  "environment": "development",
  "apnsEnvironment": "sandbox",
  "autoRequestPermission": false
]) { result, error in
  print(result as Any, error as Any)
}
```

APNs environment:

| Build type | APNs environment |
| --- | --- |
| Xcode debug or development-signed build | `sandbox` |
| TestFlight | `production` |
| App Store | `production` |
| Production-signed ad hoc build | `production` |

Register the device token:

```swift
func application(
  _ application: UIApplication,
  didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data
) {
  Sendrealm.didRegisterForRemoteNotifications(withDeviceToken: deviceToken)
}
```

Ask permission from a user action:

```swift
Sendrealm.shared.requestPermission { granted, error in
  print(granted?.boolValue == true, error as Any)
}
```

## Identity, Tags, Events

Android:

```kotlin
Sendrealm.login("user-123", "user@example.com")
Sendrealm.addTags(mapOf("plan" to "pro"))
Sendrealm.trackEvent("checkout_started", mapOf("product_id" to "sku_123"))
```

iOS:

```swift
Sendrealm.shared.login("user-123", email: "user@example.com") { error in
  print(error as Any)
}

Sendrealm.shared.addTags(["plan": "pro"]) { success, error in
  print(success.boolValue, error as Any)
}
```

Use tags for app-observed preferences and state. Use backend contact fields for authoritative account, billing, security, compliance, and verified profile data.

## Rich And Background iOS Push

- Rich image notifications require a Notification Service Extension.
- Background updates require Background Modes with Remote notifications.
- Silent/background pushes are best effort and can be throttled by iOS.

In a Notification Service Extension:

```swift
SendrealmNotificationServiceHelper.enrich(
  request: request,
  bestAttemptContent: bestAttemptContent,
  completion: contentHandler
)
```

## Diagnostics

Android:

```kotlin
val diagnostics = Sendrealm.getDiagnostics(this)
```

iOS:

```swift
Sendrealm.shared.getDiagnostics { diagnostics in
  print(diagnostics as Any)
}
```

Confirm device ID, token presence, expected permission status, expected APNs environment, subscribed state, and no unexpected SDK error.
