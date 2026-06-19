# Build Local Push Connectivity for Restricted Networks
**WWDC20 · Session 10113** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10113/)

_Platforms:_ iOS 14

## Overview
Local Push Connectivity is a new NetworkExtension API in iOS 14 that enables apps to receive push-style notifications — including VoIP calls — on Wi-Fi networks that cannot reach Apple Push Notification service (APNs). The target use cases include cruise ships, airlines, hospitals, and campuses where local Wi-Fi exists but internet connectivity is absent or restricted.

The feature consists of two components: `NEAppPushManager` (used in the containing app to configure which Wi-Fi SSIDs activate local push) and `NEAppPushProvider` (an app extension that maintains a persistent connection to the app's own provider server and dispatches notifications). The extension runs in the background for the duration of the device's association with the configured Wi-Fi networks.

A demo "SimplePush" app running on a simulated cruise ship network shows text messaging notifications via `UserNotifications` and VoIP call delivery via `CallKit`, all without any internet connection.

## Key Topics

**Architecture**
The app extension (`NEAppPushProvider` subclass) is started by the system when the device joins a configured SSID, maintains a connection to the developer's own provider server using a developer-defined protocol, and is stopped when the device leaves the network. There is no APNs involved; the communication channel is entirely between the extension and the local provider server.

**Notification Types**
- Text/alert notifications: the extension uses `UserNotifications` (local notifications) to present banners, sounds, or badges.
- VoIP notifications: the extension calls `NEAppPushProvider.reportIncomingCall(userInfo:)`. The system launches the containing app, which uses `CallKit` to display the native incoming call UI. The containing app must have the "Voice over IP" background mode enabled.

**NEAppPushManager (Containing App)**
Manages configurations (create, save, load, remove). Key properties: `matchSSIDs` (array of Wi-Fi SSIDs), `providerBundleIdentifier` (extension bundle ID), `providerConfiguration` (custom `[String: Any]` dictionary), `isEnabled`. Set a `NEAppPushDelegate` to receive incoming VoIP call notifications.

**NEAppPushProvider (App Extension)**
Subclass this abstract class. Override `start(completionHandler:)` to connect to the provider server, and `stop(with:completionHandler:)` to disconnect. Call `reportIncomingCall(userInfo:)` when a VoIP notification is received.

**Entitlement Requirement**
Both the containing app and app extension require the `NEAppPushProvider` entitlement (restricted; must be requested from Apple). This is not available to all developers without explicit approval.

## APIs & Frameworks

### NetworkExtension **[NEW]**
- `NEAppPushManager` **[NEW]** — manages local push configurations in the containing app
  - `matchSSIDs: [String]` **[NEW]** — Wi-Fi SSIDs that activate the extension
  - `providerBundleIdentifier: String` **[NEW]** — bundle ID of the app extension
  - `providerConfiguration: [String: Any]` **[NEW]** — custom config passed to the extension
  - `isEnabled: Bool` **[NEW]** — enables/disables this configuration
  - `saveToPreferences(completionHandler:)` **[NEW]** — saves and activates configuration
  - `loadAllFromPreferences(completionHandler:)` **[NEW]** — loads all saved configurations
  - `delegate: NEAppPushDelegate?` **[NEW]** — receives incoming VoIP call info
- `NEAppPushDelegate` protocol **[NEW]**
  - `appPushManager(_:didReceiveIncomingCallWithUserInfo:)` **[NEW]** — called on main queue when VoIP call info arrives
- `NEAppPushProvider` **[NEW]** — abstract base class for the app extension
  - `start(completionHandler:)` **[NEW]** — called when device joins configured SSID
  - `stop(with:completionHandler:)` **[NEW]** — called when device leaves configured SSID
  - `reportIncomingCall(userInfo:)` **[NEW]** — reports incoming VoIP call to the system; triggers app launch
- `NEProviderStopReason` — enum indicating why the extension was stopped

### UserNotifications
- `UNUserNotificationCenter` — schedule local notifications for text/alert notifications
- `UNMutableNotificationContent` — notification content (title, body, sound, badge)
- `UNNotificationRequest` — schedules the local notification

### CallKit
- `CXProvider` — reports incoming calls to the system (used in the containing app after receiving VoIP call info)
- `CXCallUpdate` — describes the incoming call (caller name, handle, etc.)
- `CXProvider.reportNewIncomingCall(with:update:completion:)` — triggers the native call UI

### UIKit / App Lifecycle
- `UIApplicationDelegate` — containing app's delegate conforming also to `NEAppPushDelegate`
- `UIApplication.LaunchOptionsKey` — handles app launch from VoIP call
- Background mode: `voip` (Voice over IP) — must be enabled in Xcode capabilities

## Code Highlights

Creating and saving a local push configuration:
```swift
import NetworkExtension

let manager = NEAppPushManager()
manager.matchSSIDs = ["Cruise Ship Wi-Fi", "Cruise Ship Staff Wi-Fi"]
manager.providerBundleIdentifier = "com.myexample.SimplePush.Provider"
manager.providerConfiguration = ["host": "cruiseship.example.com"]
manager.isEnabled = true
manager.saveToPreferences { error in
    if let error = error { /* handle */ return }
    // Configuration active
}
```

App extension lifecycle and VoIP call reporting:
```swift
class SimplePushProvider: NEAppPushProvider {
    override func start(completionHandler: @escaping (Error?) -> Void) {
        // Connect to provider server
        completionHandler(nil)
    }
    override func stop(with reason: NEProviderStopReason,
                       completionHandler: @escaping () -> Void) {
        // Disconnect from provider server
        completionHandler()
    }
    func handleIncomingVoIPCall(callInfo: [AnyHashable: Any]) {
        reportIncomingCall(userInfo: callInfo)
    }
}
```

Handling incoming VoIP call in the containing app:
```swift
class AppDelegate: UIResponder, UIApplicationDelegate, NEAppPushDelegate {
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        NEAppPushManager.loadAllFromPreferences { managers, error in
            for manager in managers ?? [] { manager.delegate = self }
        }
        return true
    }
    func appPushManager(_ manager: NEAppPushManager,
                        didReceiveIncomingCallWithUserInfo userInfo: [AnyHashable: Any]) {
        // Report to CallKit to display call UI
    }
}
```

## Takeaways
- Local Push Connectivity is a niche NetworkExtension API for environments without internet/APNs access (cruise ships, hospitals, etc.); use PushKit or UserNotifications for all other scenarios.
- The `NEAppPushProvider` extension runs only when the device is connected to the configured SSIDs, maintaining a persistent connection to the developer's own local provider server via a custom protocol.
- VoIP calls are reported via `reportIncomingCall(userInfo:)` which triggers system-level app launch and CallKit call UI; text notifications use standard `UserNotifications` local scheduling.
- The `NEAppPushProvider` entitlement is restricted and must be requested from Apple before implementation.

---
_Source: WWDC20 Session 10113 page (abstract, chapter summaries, code samples, and resource links)._
