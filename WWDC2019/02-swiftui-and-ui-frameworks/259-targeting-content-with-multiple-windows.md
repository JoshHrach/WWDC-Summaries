# Targeting Content with Multiple Windows
**WWDC19 · Session 259** · [Watch](https://developer.apple.com/videos/play/wwdc2019/259/)

_Platforms:_ iOS 13, iPadOS 13

## Overview
With iOS 13's multi-window support, an app can have several `UIScene` instances running simultaneously. This creates a new challenge: when a notification, shortcut item, or user activity arrives, which scene should activate to display its content? This session introduces `UISceneActivationConditions` and Target Content Identifiers — a system that lets the OS match incoming events to the correct scene without launching or querying the app at all, ensuring immediate, seamless scene selection.

## Key Topics

### The Multi-Window Targeting Problem
- In iOS 12, there was a 1-to-1 app-to-UI mapping: any notification simply went to the single running UI.
- In iOS 13/iPadOS 13, an app may have multiple concurrent scenes (windows). When a notification arrives, the system must pick the right scene to bring forward.
- Requiring the app to be running and queried first would introduce unacceptable latency — the user would see no response when tapping a notification.
- Solution: scenes advertise their content capabilities to the OS in advance; the OS matches incoming target content identifiers against those capabilities without touching the app process.

### Target Content Identifiers **[NEW]**
- A **target content identifier** is a structured string (ideally a URL or universal link format) that represents a specific piece of content in the app's data model.
- The identifier is attached to events (notifications, shortcut items, user activities) so the OS knows what content the event pertains to.
- The OS evaluates the identifier against each scene's activation conditions to select the best scene.
- Apps already using universal links can reuse those URL strings directly as target content identifiers.

### UISceneActivationConditions **[NEW]**
- `UISceneActivationConditions` — an object attached to every `UIScene` via `scene.activationConditions`. **[NEW]**
- Contains two `NSPredicate` properties:
  - `canActivateForTargetContentIdentifierPredicate` — expresses what content the scene is **capable** of showing. The system won't activate the scene for content that fails this predicate.
  - `prefersToActivateForTargetContentIdentifierPredicate` — expresses what content this scene **prefers** to show. Among all capable scenes, the one with the strongest preference wins.
- Default value for new scenes: can = always true (any content), prefers = always false (no preference) — appropriate for a "main" or "home" scene.
- For a detail scene open on a specific item: set both `can` and `prefers` to the predicate for that item's identifier — this is the most common pattern.
- For a multi-tab document scene (e.g., Safari): `can` = always true; `prefers` = compound `OR` predicate of all open document identifiers.

### NSPredicate Restrictions in Activation Conditions
Because predicates are shipped out of process to the OS, three restrictions apply:
1. **Block-based predicates not allowed** — code cannot be serialized and shipped.
2. **Regular expression predicates not allowed** — would introduce unbounded evaluation time.
   - Use the `LIKE` operator instead (NSPredicate's glob operator) for wildcard matching.
3. **Only `SELF` keypath allowed** — predicates operate on the target content identifier string directly; do not use keypaths like `length`.

### Where Target Content Identifiers Are Set **[NEW]**
- **Push notifications:** `UNNotificationContent.targetContentIdentifier` — set on the server in the JSON payload as the `"target-content-id"` key before sending to APNs. **[NEW]**
- **Home screen quick actions:** `UIApplicationShortcutItem.targetContentIdentifier` — supported on iPadOS 13 (long-press icon for quick actions). **[NEW]**
- **NSUserActivity:** `NSUserActivity.targetContentIdentifier` — used for state restoration, Handoff, Spotlight continuations. **[NEW]**

### Practical Architecture Patterns
- **Main/home scene:** Leave activation conditions at default (can = any, prefers = none). The main scene is the fallback for anything no detail scene claims.
- **Detail scene for a specific item:** Set `can` and `prefers` both to `NSPredicate(format: "SELF == %@", itemURL)`.
- **Multi-document scene (tabs):** `can` = `NSPredicate(value: true)`; `prefers` = `NSCompoundPredicate(orPredicateWithSubpredicates: [p1, p2, ...])` for all open documents.
- Update `prefersToActivateForTargetContentIdentifierPredicate` whenever the user opens or closes a document in a scene.

## APIs & Frameworks

### UIKit — Scene Content Targeting **[NEW]**
- `UIScene.activationConditions: UISceneActivationConditions` — access scene activation conditions **[NEW]**
- `UISceneActivationConditions` — object holding can/prefers predicates **[NEW]**
  - `canActivateForTargetContentIdentifierPredicate: NSPredicate` — capability predicate **[NEW]**
  - `prefersToActivateForTargetContentIdentifierPredicate: NSPredicate` — preference predicate **[NEW]**
- `UIApplicationShortcutItem.targetContentIdentifier: String?` — target identifier for quick actions **[NEW]**

### UserNotifications **[NEW]**
- `UNNotificationContent.targetContentIdentifier: String?` — target identifier from APNs payload **[NEW]**
- APNs payload key: `"target-content-id"` — set on the server, received by all scenes for matching

### Foundation
- `NSUserActivity.targetContentIdentifier: String?` — target identifier for Handoff/state restoration **[NEW]**
- `NSPredicate(format:)` — string comparison predicates for activation conditions
- `NSPredicate(value: true)` — always-true predicate (default can predicate)
- `NSCompoundPredicate(orPredicateWithSubpredicates:)` — combine multiple document predicates

## Code Highlights

Setting activation conditions for a detail scene showing a specific clown:
```swift
// In UISceneDelegate.scene(_:willConnectTo:options:)
let crustyURL = "https://clowntown.example.com/clowns/crusty"
let predicate = NSPredicate(format: "SELF == %@", crustyURL)
scene.activationConditions.canActivateForTargetContentIdentifierPredicate = predicate
scene.activationConditions.prefersToActivateForTargetContentIdentifierPredicate = predicate
```

Setting activation conditions for a main scene (default, accepts any content):
```swift
// Default — no code needed, but explicit form:
scene.activationConditions.canActivateForTargetContentIdentifierPredicate = NSPredicate(value: true)
scene.activationConditions.prefersToActivateForTargetContentIdentifierPredicate = NSPredicate(value: false)
```

Updating preferences when a tab is opened/closed (multi-document scene):
```swift
func updateActivationConditions() {
    let subpredicates = openDocumentURLs.map { url in
        NSPredicate(format: "SELF == %@", url)
    }
    scene.activationConditions.prefersToActivateForTargetContentIdentifierPredicate =
        NSCompoundPredicate(orPredicateWithSubpredicates: subpredicates)
}
```

Annotating a push notification payload (server-side JSON):
```json
{
  "aps": {
    "alert": "Your clown Crusty has arrived!",
    "target-content-id": "https://clowntown.example.com/clowns/crusty"
  }
}
```

Annotating a home screen quick action:
```swift
let shortcut = UIApplicationShortcutItem(
    type: "ViewCrusty",
    localizedTitle: "View Crusty",
    localizedSubtitle: nil,
    icon: UIApplicationShortcutIcon(type: .search),
    userInfo: nil
)
// New in iOS 13 / iPadOS 13:
shortcut.targetContentIdentifier = "https://clowntown.example.com/clowns/crusty"
```

## Takeaways
- In a multi-window app, every scene must declare its content capabilities via `UISceneActivationConditions` so the OS can route notifications and shortcuts to the right scene without running the app.
- Target content identifiers should be structured strings like universal links — reuse your existing URL scheme if you have one.
- The `LIKE` operator in NSPredicate is the right tool for wildcard/glob matching in activation conditions; block-based and regex predicates are forbidden.
- Update `prefersToActivateForTargetContentIdentifierPredicate` whenever the set of content shown in a scene changes (tab opens/closes, navigation changes).

---
_Source: WWDC19 Session 259 page (abstract, full transcript, and resource links)._
