# Support multiple users in your tvOS app
**WWDC20 · Session 10645** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10645/)

_Platforms:_ tvOS 14

## Overview
tvOS 14 introduces the **Runs as Current User** capability, which allows tvOS apps to maintain completely separate data, iCloud storage, Game Center profiles, and local preferences for each person who uses Apple TV in a household. When enabled, the system automatically switches the app's storage context when a user switches in Control Center — requiring only a single Xcode capability checkbox and two optional lifecycle handlers, with no changes to existing data storage code.

The key insight is that all standard Apple data storage APIs (UserDefaults, Core Data, NSPersistentCloudKitContainer, CloudKit, Game Center) already support per-user isolation at the system level. Enabling the "Runs as Current User" capability is the only step needed to activate this isolation; no data-layer changes are required. The app is terminated and relaunched automatically for the new user whenever a switch occurs.

## Key Topics

### Runs as Current User Capability (NEW)
- Added in tvOS 14 / Xcode 12 under the **User Management** capability in the Signing & Capabilities editor
- When enabled, the app's process runs in the context of the currently active tvOS user account
- All data APIs (iCloud, CloudKit, NSPersistentCloudKitContainer, Game Center, UserDefaults, file system sandbox) automatically scope to that user — zero code changes needed
- Replaces the need to build an in-app profile picker for apps that do not have server-side user accounts

### App Lifecycle on User Switch
- When a user switch occurs via Control Center, the foreground app receives `applicationWillTerminate` and is given a limited time window to save pending changes
- After termination, if the app was in the foreground, it is automatically relaunched for the new user
- The process does **not** receive `applicationDidEnterBackground` before termination on a user switch — only `applicationWillTerminate`

### CloudKit Notification Filtering (NEW)
- When an app supports multiple users, it can receive CloudKit push notifications originally intended for **other** users on the same device (because the device is registered for the subscription across all accounts)
- Filter incoming notifications by comparing `CKNotification.subscriptionOwnerUserRecordID` (new property) against the current user's CloudKit record ID
- Discard notifications whose `subscriptionOwnerUserRecordID` does not match the active user's record ID

### Relationship to TVUserManager
- `TVUserManager` (tvOS 13) — for apps with their own server-side user profiles; links in-app profiles to tvOS user accounts
- "Runs as Current User" — for apps without their own profiles; the OS provides per-user data isolation automatically
- These are complementary, not mutually exclusive

## APIs & Frameworks

**TVServices / User Management**
- `Runs as Current User` capability — Xcode Signing & Capabilities → User Management **[NEW in tvOS 14]**
- `TVUserManager` (tvOS 13) — links in-app profiles to tvOS user accounts (complementary, not required for this feature)

**UIKit Application Lifecycle**
- `UIApplicationDelegate.applicationWillTerminate(_:)` — called when a user switch is about to terminate the app; save pending state here
  - Time is limited — do minimal synchronous or semaphore-guarded async work
  - Check for unsaved changes before doing any work to return quickly if none exist

**CloudKit**
- `CKNotification(fromRemoteNotificationDictionary:)` — parse a push notification dictionary into a typed notification
- `CKNotification.subscriptionOwnerUserRecordID: CKRecord.ID?` **[NEW]** — the CloudKit user record ID of the account that owns the subscription that triggered the notification
- `CKContainer.currentUserRecordID` / `fetchCurrentUserRecordID(completionHandler:)` — get the active user's record ID for comparison
- `NSPersistentCloudKitContainer` — Core Data + CloudKit sync; automatically uses the active user's iCloud account when "Runs as Current User" is enabled

**Game Center**
- Per-user Game Center accounts (achievements, leaderboards, friends) are automatically scoped when "Runs as Current User" is enabled

**Foundation / Core Data**
- `UserDefaults.standard` — automatically per-user when capability is enabled
- `NSPersistentContainer` / `NSPersistentCloudKitContainer` — local and cloud-synced persistent stores are automatically per-user
- File system sandbox — each user gets their own app container

## Code Highlights

Save pending data before process termination on user switch:
```swift
func applicationWillTerminate(_ application: UIApplication) {
    // Return early if nothing to save — don't delay the user switch
    guard game.hasUnsavedChanges else { return }

    let semaphore = DispatchSemaphore(value: 0)
    game.save { _ in semaphore.signal() }
    semaphore.wait()  // keep the process alive until async save completes
}
```

Filter CloudKit notifications to avoid handling another user's events:
```swift
func application(
    _ application: UIApplication,
    didReceiveRemoteNotification userInfo: [AnyHashable: Any],
    fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void
) {
    guard let notification = CKNotification(fromRemoteNotificationDictionary: userInfo),
          notification.subscriptionOwnerUserRecordID == game.currentUserRecordID else {
        completionHandler(.noData)
        return
    }
    game.handle(notification, completionHandler: completionHandler)
}
```

## Takeaways

- Enable "Runs as Current User" in the User Management capability to give every household member their own iCloud, Game Center, local storage, and preferences within your app — no data-layer code changes are required.
- Implement `applicationWillTerminate` to save any unsaved state before the system terminates the app on a user switch; return immediately if there are no pending changes to keep the switch fast.
- Filter incoming CloudKit push notifications by comparing `CKNotification.subscriptionOwnerUserRecordID` against the current user's record ID so cross-user notifications are safely discarded.
- "Runs as Current User" is complementary to `TVUserManager` — use `TVUserManager` when your app has its own server-side accounts; use "Runs as Current User" when you want the OS to provide per-user data isolation automatically.

---
_Source: WWDC20 Session 10645 page (transcript and code samples)._
