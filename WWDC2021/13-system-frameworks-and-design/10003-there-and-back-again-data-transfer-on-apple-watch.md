# There and Back Again: Data Transfer on Apple Watch
**WWDC21 · Session 10003** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10003/)

_Platforms:_ watchOS 8, iOS 15

## Overview
This session covers the full spectrum of data transfer strategies available for Apple Watch apps in watchOS 8, including support for the increasingly independent Watch (Family Setup, standalone apps). The session provides a decision framework for choosing the right data transfer mechanism based on data type, source/destination, companion dependency, Family Setup support needs, and timing requirements.

Four main categories are covered: iCloud (Keychain with synchronization, Core Data with CloudKit), Watch Connectivity (for paired iPhone/Watch communication), URL sessions (background and foreground for direct server communication), and sockets (for streaming audio). Each approach is compared for battery efficiency, latency, Family Setup compatibility, and companion iPhone dependency.

The session includes detailed code for storing, updating, retrieving, and deleting Keychain items with iCloud synchronization, a full background URL session implementation with WKExtensionDelegate integration, and Core Data/SwiftUI patterns for watchOS.

## Key Topics

### Decision Framework
- What kind of data is it? Where is it now and where does it need to go?
- Does the interaction depend on a companion iOS app?
- Should the app support Family Setup (no companion iPhone required)?
- Can data transfer be deferred for system optimization?
- How frequently does the data change?

### Keychain with iCloud Synchronization
- Available since watchOS 6.2; syncs to all of a user's devices
- Two approaches: Password AutoFill with Associated Domains, or direct Keychain sharing
- `kSecAttrSynchronizable: true` required to mark items for iCloud sync
- `kSecClassInternetPassword` for credential storage
- `SecItemAdd`, `SecItemUpdate`, `SecItemCopyMatching`, `SecItemDelete` for CRUD operations
- Supports Family Setup; does not require companion iPhone
- Customers can disable iCloud Keychain; not available in all regions

### Core Data with CloudKit
- Syncs local database to all devices sharing the same CloudKit container
- SwiftUI integration: `@Environment(\.managedObjectContext)`, `@FetchRequest` property wrapper
- Avoid sending too much data to Watch; use multiple Core Data model configurations to segment data
- Supports Family Setup; no companion iPhone required
- Synchronization timing based on network and system conditions

### Watch Connectivity
- Communicates between Watch app and its paired companion iPhone app (Bluetooth or same Wi-Fi)
- `WCSession`: activate as early as possible (in app/extension delegate `applicationDidFinishLaunching`)
- All delegate callbacks are on a non-main serial queue — dispatch UI updates to main queue
- **Application Context** (`updateApplicationContext`): single dictionary, replaced each time; good for frequently-updated state
- **User Info Transfer** (`transferUserInfo`): queued dictionaries, delivered in order; can cancel via queue
- **File Transfer** (`transferFile`): queued files; move or process immediately in `didReceiveFile` (file deleted after callback)
- **`transferCurrentComplicationUserInfo(_:)`**: prioritized transfer for complication data; counts against complication budget (up to 4 transfers/hour with active complication)
- **`sendMessage(_:replyHandler:errorHandler:)`** / **`sendMessageData(_:replyHandler:errorHandler:)`**: interactive messaging requiring reachability
- `WCSession.isReachable`: Watch Extension must be in foreground (or active background session); iOS app is reachable from Watch far more often
- Not suitable for Family Setup apps

### Background URL Sessions
- `URLSessionConfiguration.background(withIdentifier:)`
- `sessionSendsLaunchEvents = true`: app launched in background when tasks complete
- `isDiscretionary = true`: for large transfers; system schedules at optimal time
- `task.earliestBeginDate`: schedule future start; system respects background budget/network conditions
- Apps receive up to 4 background refresh tasks per hour with active complication; schedule at least 15 minutes apart
- `WKURLSessionRefreshBackgroundTask`: handled in `WKExtensionDelegate.handle(_:)`
- Always call `setTaskCompletedWithSnapshot(false)` when done; failure causes system termination
- `@WKExtensionDelegateAdaptor` property wrapper to connect extension delegate to SwiftUI app
- Supports Family Setup; no companion iPhone required

### Foreground URL Sessions
- `URLSessionConfiguration.default`
- For quick, interactive server communication (e.g., latest workout list, daily content)
- Less power-efficient than background sessions; 2.5-minute timeout enforced
- Should target interactions much shorter than the 2.5-minute limit

### Sockets
- HTTP Live Streaming and WebSockets available in Watch apps during active audio streaming sessions
- Use for streaming audio apps only

## APIs & Frameworks

- `Security` framework
- `SecItemAdd(_:_:)`, `SecItemUpdate(_:_:)`, `SecItemCopyMatching(_:_:)`, `SecItemDelete(_:)`
- `kSecClass`, `kSecClassInternetPassword`
- `kSecAttrServer`, `kSecAttrAccount`, `kSecAttrSynchronizable`
- `kSecValueData`, `kSecReturnData`, `kSecReturnAttributes`
- `errSecItemNotFound`, `errSecSuccess`
- `CoreData` framework
- `NSPersistentCloudKitContainer`
- `@FetchRequest` property wrapper (SwiftUI)
- `@Environment(\.managedObjectContext)`
- `WatchConnectivity` framework
- `WCSession`
- `WCSession.default`
- `WCSessionDelegate`
- `WCSession.isReachable`
- `WCSession.applicationContext`
- `WCSession.updateApplicationContext(_:)`
- `WCSession.transferUserInfo(_:)` → `WCSessionUserInfoTransfer`
- `WCSession.transferFile(_:metadata:)` → `WCSessionFileTransfer`
- `WCSession.transferCurrentComplicationUserInfo(_:)`
- `WCSession.sendMessage(_:replyHandler:errorHandler:)`
- `WCSession.sendMessageData(_:replyHandler:errorHandler:)`
- `WCSessionDelegate.session(_:didReceiveApplicationContext:)`
- `WCSessionDelegate.session(_:didReceiveUserInfo:)`
- `WCSessionDelegate.session(_:didReceive:)`
- `WCSessionDelegate.session(_:didReceiveMessage:replyHandler:)`
- `Foundation` framework
- `URLSession`
- `URLSessionConfiguration.background(withIdentifier:)`
- `URLSessionConfiguration.isDiscretionary`
- `URLSessionConfiguration.sessionSendsLaunchEvents`
- `URLSessionDownloadTask.earliestBeginDate`
- `URLSessionDownloadDelegate.urlSession(_:downloadTask:didFinishDownloadingTo:)`
- `WatchKit` framework
- `WKExtensionDelegate`
- `WKExtensionDelegate.handle(_:)`
- `WKURLSessionRefreshBackgroundTask`
- `WKURLSessionRefreshBackgroundTask.setTaskCompletedWithSnapshot(_:)`
- `WKApplicationRefreshBackgroundTask`
- `WKSnapshotRefreshBackgroundTask`
- `WKWatchConnectivityRefreshBackgroundTask`
- `@WKExtensionDelegateAdaptor` **[NEW]** (SwiftUI app integration for extension delegate)
- Associated Domains capability (for Password AutoFill)
- Keychain Sharing capability / App Groups capability

## Code Highlights

Storing an OAuth 2 token in the Keychain with iCloud sync:
```swift
let query: [String: Any] = [
    kSecClass as String: kSecClassInternetPassword,
    kSecAttrServer as String: server,
    kSecAttrAccount as String: account,
    kSecAttrSynchronizable as String: true,
]
let attributes: [String: Any] = [kSecValueData as String: tokenData]
let status = SecItemUpdate(query as CFDictionary, attributes as CFDictionary)
```

Core Data view with SwiftUI and `@FetchRequest`:
```swift
@Environment(\.managedObjectContext) private var viewContext
@FetchRequest(sortDescriptors: [NSSortDescriptor(keyPath: \Setting.itemKey, ascending: true)])
private var settings: FetchedResults<Setting>
```

Connecting SwiftUI app to WKExtensionDelegate:
```swift
@main struct MyWatchApp: App {
    @WKExtensionDelegateAdaptor(ExtensionDelegate.self) var extensionDelegate
    // ...
}
```

## Takeaways

- Use background URL sessions whenever possible; foreground sessions have a hard 2.5-minute limit and are less power-efficient.
- Watch Connectivity is ideal for optimizing the paired iPhone+Watch experience but cannot be used for Family Setup; always activate `WCSession` at app launch.
- Keychain with `kSecAttrSynchronizable: true` and Core Data + CloudKit are the right choices when apps must support Family Setup (no paired iPhone).
- Always call `setTaskCompletedWithSnapshot(false)` on background refresh tasks promptly; failure results in app termination by the system.

---
_Source: WWDC21 Session 10003 page (abstract, chapter summaries, code samples, and resource links)._
