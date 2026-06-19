# Adopt Quick Note
**WWDC21 · Session 10264** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10264/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
Quick Note is a new system-wide note-taking feature in iPadOS 15 and macOS Monterey that lets users bring up a note over any app — with an Apple Pencil swipe from the corner, a keyboard shortcut, or a control — and link app content directly into that note. Apps adopt Quick Note through `NSUserActivity`, the same API used for Handoff, Spotlight, and state restoration, so there is minimal incremental adoption work.

Once an app's content is linked in a Quick Note, Quick Note Suggestions automatically appear when the user revisits that content in the app or on the web — showing a floating chip that taps back to the original note. The feature works cross-platform: a link created on iPad appears as a suggestion on Mac and vice versa.

## Key Topics

### How Quick Note Works
- Registered `NSUserActivity` objects are sent to the system and consumed by Handoff, Spotlight, Reminders, and now Quick Note
- Activities with qualifying identifiers appear in Quick Note's "Add Link" menu
- When the user returns to linked content, Quick Note Suggestions appear in the bottom-right corner
- Works on iPad (Apple Pencil swipe up from bottom-right), and macOS Monterey

### Required NSUserActivity Properties for Quick Note
Set at least one of these stable, globally unique identifiers:
- `targetContentIdentifier` — also used for state restoration and multitasking on iPad
- `persistentIdentifier` — also used for Spotlight indexing
- `webpageURL` — also used for Handoff web fallback

Identifiers must be: unique per content item, global (not device-specific), and stable over time.

### Adoption Steps
1. Declare activity types in `NSUserActivityTypes` key in `Info.plist`
2. Create `NSUserActivity` objects with `title`, identifier(s), and `userInfo`
3. Attach activities to responders (`viewController.userActivity = activity`) — UIKit/AppKit manages currency automatically
4. Implement continuation handlers in `UIWindowSceneDelegate` (iOS) or `NSApplicationDelegate` (macOS)

### Best Practices
- Use document/item titles for `activity.title` (shown in the Add Link menu)
- Never use device-specific paths, session IDs, or mutable properties as identifiers — use saved UUIDs
- Use `needsSave = true` + `userActivityWillSave(_:)` delegate for expensive-to-compute state (e.g., scroll position)
- Do not reuse activity instances; create a new `NSUserActivity` when content changes
- Use consistent values for `targetContentIdentifier` and `persistentIdentifier` if supporting both state restoration and Spotlight
- Prepare for version compatibility: handle activities from older/newer app versions gracefully
- Handle missing/deleted content: show error if deleted, redirect if moved

## APIs & Frameworks

### Foundation
- `NSUserActivity` — core activity object
  - `activityType: String` — declared in `Info.plist` `NSUserActivityTypes`
  - `title: String` — displayed in Quick Note Add Link menu
  - `targetContentIdentifier: String?` — unique stable ID; used for Quick Note, state restoration, multitasking
  - `persistentIdentifier: String?` — unique stable ID; used for Quick Note and Spotlight
  - `webpageURL: URL?` — URL identifier; used for Quick Note and Handoff web fallback
  - `userInfo: [AnyHashable: Any]?` — app-specific state for restoration
  - `needsSave: Bool` — defer userInfo updates until system actually needs the activity
  - `becomeCurrent()` / `resignCurrent()` — manual current-activity management
- `NSUserActivityDelegate`
  - `userActivityWillSave(_:)` — update userInfo on demand when `needsSave` is set

### UIKit
- `UIResponder.userActivity: NSUserActivity?` — attach activity to responder for automatic management
- `UIWindowSceneDelegate`
  - `scene(_:willContinueUserActivityWithType:)` — show loading UI
  - `scene(_:continue:)` — restore app state from activity
  - `scene(_:didFailToContinueUserActivityWithType:error:)` — handle errors

### AppKit (macOS)
- `NSApplicationDelegate`
  - `application(_:willContinueUserActivityWithType:) -> Bool`
  - `application(_:continue:restorationHandler:) -> Bool`
  - `application(_:didFailToContinueUserActivityWithType:error:)`

## Code Highlights

Creating and attaching an `NSUserActivity`:
```swift
let activity = NSUserActivity(activityType: "com.myapp.MyActivityType")
activity.title = document.title
activity.targetContentIdentifier = "uniqueGlobalStableIdentifier"
activity.userInfo = ["myKey": someValue]
viewController.userActivity = activity
```

Deferred state update with `needsSave`:
```swift
activity.needsSave = true
// Later, called by system when needed:
func userActivityWillSave(_ userActivity: NSUserActivity) {
    userActivity.userInfo = [
        "center": visibleFrame.middle,
        "zoomScale": scrollView.zoomScale
    ]
}
```

Handling continuation on iOS:
```swift
func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
    // Set up view controllers and restore state from userActivity.userInfo
}
```

## Takeaways
- Quick Note adoption is essentially free if you already have `NSUserActivity` set up for Handoff or Spotlight — just ensure the three identifier properties are populated with stable, global, unique values.
- The `needsSave` pattern is important for performance: don't update userInfo on every gesture; let the system pull it when needed.
- Attaching activities to UIKit/AppKit responders (rather than calling `becomeCurrent()` manually) is the recommended approach — the system manages the current activity for you.
- Quick Note gives apps a new discoverability surface: content linked from your app appears in Notes across all the user's devices.

---
_Source: WWDC21 Session 10264 page (abstract, chapter summaries, code samples, and resource links)._
