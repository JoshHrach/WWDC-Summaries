# Creating Independent Watch Apps
**WWDC19 · Session 208** · [Watch](https://developer.apple.com/videos/play/wwdc2019/208/)

_Platforms:_ watchOS 6, iOS 13

## Overview
watchOS 6 introduces the concept of fully independent watch apps — apps that no longer require an iOS companion to be installed or even to exist. For the first time, developers can ship a watch-only app without any iOS counterpart. The App Store now exists natively on Apple Watch, allowing users to discover, purchase, and install apps directly from their wrist.

The session covers the three pillars of independent app architecture: authentication and user data acquisition directly on the watch, push notifications targeting Apple Watch as a standalone device, and networking and CloudKit data pipelines that bypass the iPhone entirely. Debugging speed improvements (up to 10x faster in Simulator, 2x faster on device) and asset thinning for Watch apps round out the engineering improvements.

## Key Topics

- **Independent vs. dependent apps** — A "dependent" Watch app requires the companion iOS app to be installed before it can launch. An "independent" Watch app can run without any iOS app. Existing apps become independent by checking "Supports Running Without iOS App Installation" in the WatchKit Extension target. Watch-only apps (no iOS app at all) are also supported for the first time.
- **App Store on Apple Watch** — A fully featured App Store with featured sections, search, editorial, and in-app purchase, all from the wrist. **[NEW]**
- **Installation architecture change** — Watch apps are no longer embedded inside the iOS app binary. The App Store server performs per-device installs using bitcode recompilation, enabling asset and architecture variant thinning for Watch apps for the first time.
- **Authentication** — Sign In with Apple, text-field-based custom credential sign-in, associated domains for password AutoFill, Continuity Keyboard for text entry, and one-time-code AutoFill all available on watchOS 6.
- **Direct push notifications to Apple Watch** — Apple Watch is now a standalone APNS push target. Supports alert and background push types via the same `UNUserNotificationCenter` + `WKExtension` infrastructure used on other platforms.
- **Complication pushes via PushKit** — `PKPushRegistry` is now available on watchOS for complication push registration and delivery, replacing the old iOS-proxied mechanism. **[NEW]**
- **Networking and CloudKit** — `URLSession` (including background sessions) and CloudKit (`CKSubscription`) now fully replace Watch Connectivity for independent apps.
- **Health authorization on-device** — HealthKit authorization can now be requested directly on Apple Watch without requiring iPhone. **[NEW]**
- **Simulator debugging** — Up to 10x faster for Simulator; 2x faster for physical device debugging; Wi-Fi debugging route automatically chosen when available.

## APIs & Frameworks

### WatchKit
- `WKExtensionDelegate` — `applicationDidFinishLaunching`, `didReceiveRemoteNotification(withCompletionHandler:)` **[background notifications NEW]**
- `WKExtension.shared().registerForRemoteNotifications()` — registers Watch app for APNS **[NEW]**
- `WKAlertAction` — used for terms and conditions acceptance flows
- `isCompanionAppInstalled` property on `WKExtension` — check if paired iOS app is present **[NEW]**

### AuthenticationServices (now on watchOS)
- `ASAuthorizationAppleIDButton` — Sign In with Apple UI button **[NEW on watchOS]**
- `ASAuthorizationController` — presents Sign In with Apple sheet **[NEW on watchOS]**
- Associated domains (entitlement) on WatchKit Extension — elevates iCloud Keychain AutoFill suggestions

### UserNotifications
- `UNUserNotificationCenter.requestAuthorization(options:)` — request alert notification permission
- `UNNotificationServiceExtension` — now supported on watchOS for payload decryption **[NEW]**

### PushKit (now on watchOS)
- `PKPushRegistry` — register for complication pushes **[NEW on watchOS]**
- `PKPushRegistryDelegate` — `pushRegistry(_:didUpdate:for:)`, `pushRegistry(_:didReceiveIncomingPushWith:for:)` **[NEW on watchOS]**
- `PKPushType.complication` — push type for complication updates

### APNS
- `apns-push-type` request header key — new required key; values `alert` or `background` **[NEW, required for watchOS]**
- `apns-topic` — must be the WatchKit App bundle ID (not extension)

### URLSession
- Background `URLSession` — preferred for network requests in Watch apps; ensures completion after wrist-down
- `URLSessionConfiguration.background(withIdentifier:)`

### CloudKit
- `CKSubscription` — now fully supported on watchOS **[NEW]**
- CloudKit push notifications (`shouldSendContentAvailable = true`) — deliver background pushes on `didReceiveRemoteNotification`
- `CKFetchRecordZoneChangesOperation` — retrieve only changed records after push

### HealthKit
- `HKHealthStore.requestAuthorization(toShare:read:completion:)` — now presentable directly on Apple Watch **[NEW]**

### SwiftUI / WatchKit text fields
- `TextField` (SwiftUI) / `WKInterfaceTextField` — new text input on watchOS **[NEW]**
- `textContentType` — drives AutoFill suggestions (username, password, oneTimeCode, etc.)
- Continuity Keyboard — prompts paired iPhone/iPad for keyboard input **[NEW]**

## Code Highlights

Registering for remote notifications on watchOS:

```swift
// In WKExtensionDelegate.applicationDidFinishLaunching
UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .sound, .badge]) { granted, _ in
    guard granted else { return }
    WKExtension.shared().registerForRemoteNotifications()
}

func didRegisterForRemoteNotifications(withDeviceToken deviceToken: Data) {
    // Forward token to your notification provider server
}

func didReceiveRemoteNotification(_ userInfo: [AnyHashable: Any],
                                  fetchCompletionHandler: @escaping (WKBackgroundFetchResult) -> Void) {
    // Handle background notification
    fetchCompletionHandler(.newData)
}
```

Registering for complication pushes via PushKit:

```swift
let registry = PKPushRegistry(queue: .main)
registry.delegate = self
registry.desiredPushTypes = [.complication]

func pushRegistry(_ registry: PKPushRegistry, didUpdate credentials: PKPushCredentials, for type: PKPushType) {
    // Forward complication push token to server
}

func pushRegistry(_ registry: PKPushRegistry, didReceiveIncomingPushWith payload: PKPushPayload, for type: PKPushType) {
    CLKComplicationServer.sharedInstance().reloadTimeline(for: complication)
}
```

## Takeaways

- Enable "Supports Running Without iOS App Installation" in the WatchKit Extension target — a single checkbox — to make any existing Watch app independent.
- Replace Watch Connectivity data pipelines with `URLSession` background sessions and `CKSubscription` to enable fully independent operation; use `isCompanionAppInstalled` to conditionally retain companion features.
- Add the `apns-push-type` header (set to `alert` or `background`) to all APNS requests — it is required for watchOS and good practice for all Apple platforms.
- Use PushKit's `PKPushType.complication` directly on watchOS instead of routing complication pushes through iOS.

---
_Source: WWDC19 Session 208 page (abstract, full transcript, and resource links)._
