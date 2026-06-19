# The Push Notifications Primer
**WWDC20 · Session 10095** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10095/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7, tvOS 14

## Overview
This primer introduces the two core types of push notifications on Apple platforms—alert notifications and background notifications—and walks through the complete implementation of each. The session is aimed at developers who are new to push notifications or need a clear refresher on the correct APIs and patterns for each type.

Alert notifications deliver visible, user-facing banners with customizable appearance and actions. They require explicit user permission (`.alert`, `.sound`, `.badge` options via `UNUserNotificationCenter`) and must be opened via `UNUserNotificationCenterDelegate`. Background notifications are silent; they wake the app to refresh content and require only `content-available: 1` in the APNs payload, no user permission prompt. Both types require calling `UIApplication.shared.registerForRemoteNotifications()` to obtain a device token, which must be forwarded to the app's back-end push server.

A key distinction is that the `UNUserNotificationCenterDelegate` and its `didReceive(_:withCompletionHandler:)` are only used for alert notifications. Background notifications are handled entirely through `UIApplication`'s `didReceiveRemoteNotification(_:fetchCompletionHandler:)` delegate method, which takes a `UIBackgroundFetchResult` in its completion handler.

## Key Topics
- **Alert vs. background notifications** — alert = visible user-facing; background = silent, wakes app to update content
- **Registration flow** — `registerForRemoteNotifications()` → `didRegisterForRemoteNotificationsWithDeviceToken` → forward token to server
- **Token conversion** — `Data` → hex string using `map { String(format: "%02.2hhx", $0) }.joined()`
- **User authorization** — `UNUserNotificationCenter.requestAuthorization(options:)` only for alert notifications; request in context
- **APNs alert payload** — `aps.alert.title`, `aps.alert.body`, `aps.sound`, `aps.badge`; custom data outside `aps`
- **Handling alert notification taps** — `UNUserNotificationCenterDelegate.userNotificationCenter(_:didReceive:withCompletionHandler:)`; always call completionHandler
- **APNs background payload** — only requires `aps.content-available: 1`; custom data outside `aps`
- **Handling background notifications** — `UIApplicationDelegate.application(_:didReceiveRemoteNotification:fetchCompletionHandler:)`; `UIBackgroundFetchResult`: `.newData`, `.noData`, `.failed`
- **Background constraints** — system limits daily background operations; skipped if device is in low-battery state
- **Best practice** — request authorization only in response to a user action with clear context; set badge to 0 after opening

## APIs & Frameworks

**UIKit / AppKit**
- `UIApplication.shared.registerForRemoteNotifications()` — start APNs registration; returns device token via delegate
- `UIApplicationDelegate.application(_:didRegisterForRemoteNotificationsWithDeviceToken:)` — receive device token (`Data`)
- `UIApplicationDelegate.application(_:didFailToRegisterForRemoteNotificationsWithError:)` — handle registration failure
- `UIApplicationDelegate.application(_:didReceiveRemoteNotification:fetchCompletionHandler:)` — handle background push notifications
- `UIBackgroundFetchResult` enum — `.newData`, `.noData`, `.failed`

**User Notifications framework**
- `UNUserNotificationCenter.current()` — singleton notification center
- `UNUserNotificationCenter.requestAuthorization(options:completionHandler:)` — prompt user for alert/sound/badge permission
  - `UNAuthorizationOptions`: `.alert`, `.sound`, `.badge`, `.provisional`, `.criticalAlert`
- `UNUserNotificationCenterDelegate` — protocol for handling notification interaction
- `UNUserNotificationCenter.delegate` — assign to receive interaction callbacks
- `UNUserNotificationCenterDelegate.userNotificationCenter(_:didReceive:withCompletionHandler:)` — called when user taps an alert notification; must call `completionHandler()`
- `UNNotificationResponse` — wrapper around the notification that was tapped
- `UNNotificationResponse.notification.request.content.userInfo` — `[AnyHashable: Any]` dict with APNs payload

**APNs Payload keys (JSON)**
- `aps` — required container dictionary
  - `alert` — dictionary with `title` (short), `body` (full text), optional `subtitle`, `launch-image`
  - `sound` — `"default"` or custom sound file name
  - `badge` — absolute badge count (Int); set to `0` to clear
  - `content-available` — `1` = background notification (silent push)
- Custom keys — any JSON-serializable data outside `aps` for the app to parse from `userInfo`

**URLSession** (referenced)
- `URLSession.shared.dataTask(with:completionHandler:)` — fetch data in background notification handler

## Code Highlights

Register for remote notifications and set up delegate:
```swift
class AppDelegate: UIResponder, UIApplicationDelegate, UNUserNotificationCenterDelegate {
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        UIApplication.shared.registerForRemoteNotifications()
        UNUserNotificationCenter.current().delegate = self
        return true
    }
}
```

Convert device token `Data` to hex string for server:
```swift
func forwardTokenToServer(token: Data) {
    let tokenString = token.map { String(format: "%02.2hhx", $0) }.joined()
    var urlComps = URLComponents(string: "www.example.com/register")!
    urlComps.queryItems = [URLQueryItem(name: "deviceToken", value: tokenString)]
    guard let url = urlComps.url else { return }
    URLSession.shared.dataTask(with: url).resume()
}
```

Request authorization in context:
```swift
UNUserNotificationCenter.current()
    .requestAuthorization(options: [.alert, .sound, .badge]) { granted, error in
        print("Permission granted: \(granted)")
    }
```

Handle alert notification tap:
```swift
func userNotificationCenter(_ center: UNUserNotificationCenter,
                            didReceive response: UNNotificationResponse,
                            withCompletionHandler completionHandler: @escaping () -> Void) {
    let userInfo = response.notification.request.content.userInfo
    // parse custom keys from userInfo...
    completionHandler() // always call
}
```

Background notification payload:
```json
{ "aps": { "content-available": 1 }, "myCustomKey": "myCustomData" }
```

Handle background notification:
```swift
func application(_ application: UIApplication,
                 didReceiveRemoteNotification userInfo: [AnyHashable: Any],
                 fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {
    guard let url = URL(string: "www.example.com/todays-menu") else {
        completionHandler(.failed); return
    }
    URLSession.shared.dataTask(with: url) { data, _, _ in
        guard let data = data else { completionHandler(.noData); return }
        updateMenu(withData: data)
        completionHandler(.newData)
    }.resume()
}
```

## Takeaways
- Alert notifications require user permission and `UNUserNotificationCenterDelegate`; background notifications require only `content-available: 1` and `UIApplicationDelegate.didReceiveRemoteNotification`—never mix the two patterns.
- Always forward the device token to your server in `didRegisterForRemoteNotificationsWithDeviceToken`; convert the `Data` to a hex string before serialization.
- Request notification authorization only in response to a user action with clear context—never on first launch—to maximize acceptance rates.
- Always call the completion handler in both alert and background handlers; for background handlers, pass the correct `UIBackgroundFetchResult` so the system can schedule future launches intelligently.

---
_Source: WWDC20 Session 10095 page (abstract, chapter summaries, code samples, and resource links)._
