# Make your UIKit app more flexible
**WWDC25 · Session 282** · [Watch](https://developer.apple.com/videos/play/wwdc2025/282/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, visionOS 26

## Overview
UIKit receives a significant flexibility push in iOS/iPadOS 26 centered on three themes: scene-based lifecycle adoption, multi-column layout improvements, and windowing ergonomics. The session announces that UIScene-based lifecycle will become mandatory in a future major release, making now the right time for apps still using the legacy `UIApplicationDelegate`-only model to migrate.

`UISplitViewController` gains interactive column resizing and a new inspector column, reducing the need for custom split-view implementations. `UITabBarController` adds tab groups and a `managingNavigationController` hook. New window and layout APIs address longstanding pain points around orientation locking, minimum window size, and corner-aware margins.

## Key Topics

### Scene-Based Lifecycle (Mandatory Soon)
The session explicitly states the UIScene lifecycle will be required in the next major iOS release after iOS 26. Apps must implement `UISceneDelegate` (or SwiftUI's `@main` App protocol). The single-window `UIApplicationDelegate`-only path is being deprecated. Apps should migrate to multi-scene support now.

### UISplitViewController Enhancements
Interactive column resizing lets users drag the divider between columns at runtime — opt-in per split view instance. The new inspector column (`displayMode: .oneBesideSecondaryBesideInspector`) provides a standardized third panel without custom container view logic. The new `splitViewControllerLayoutEnvironment` trait fires when the column layout changes, replacing manual size-class checks.

### UIWindowScene Windowing Controls
`UIWindowScene.WindowingControlStyle` **[NEW]** controls whether a scene shows the standard macOS/iPadOS window chrome (close/minimize/zoom) or a custom style. `UISceneSizeRestrictions` gains more fine-grained APIs for setting minimum and maximum window dimensions. `UIRequiresFullscreen` plist key is deprecated in favor of the new programmatic API.

### Layout and Orientation APIs
`layoutGuide(for: .margins(cornerAdaptation: .horizontal))` **[NEW]** provides safe-area-aware margins that account for rounded screen corners, replacing manual `safeAreaInsets` arithmetic. `prefersInterfaceOrientationLocked` **[NEW]** allows a view controller to opt out of rotation without the global `UIRequiresFullscreen` plist flag. `isInteractivelyResizing` lets code detect live drag-resize to temporarily reduce animation cost.

### UITabBarController Tab Groups
Tab groups let apps organize tabs into collapsible sections in the sidebar. `managingNavigationController` provides direct access to the navigation stack for the selected tab, simplifying deep-link handling.

## APIs & Frameworks

- **UIScene / UISceneDelegate** — mandatory lifecycle path (migration now recommended)
- **UISplitViewController** interactive column resizing **[NEW]**
- **UISplitViewController** inspector column **[NEW]**
- **splitViewControllerLayoutEnvironment** trait **[NEW]** — replaces size-class checks for split layout
- **UITabBarController** tab groups **[NEW]**
- **UITabBarController.managingNavigationController** **[NEW]**
- **UIWindowScene.WindowingControlStyle** **[NEW]** — window chrome customization
- **UISceneSizeRestrictions** — expanded min/max window size API **[NEW granularity]**
- **layoutGuide(for: .margins(cornerAdaptation: .horizontal))** **[NEW]**
- **prefersInterfaceOrientationLocked** **[NEW]** — per-view-controller orientation lock
- **isInteractivelyResizing** — live resize detection
- **UIRequiresFullscreen** (Info.plist) — deprecated

## Code Highlights

```swift
// Enable interactive column resizing on a split view controller
splitViewController.presentsWithGesture = true
splitViewController.preferredDisplayMode = .oneBesideSecondary

// Add inspector column
splitViewController.preferredDisplayMode = .oneBesideSecondaryBesideInspector
```

```swift
// Lock orientation per view controller (replaces UIRequiresFullscreen)
override var prefersInterfaceOrientationLocked: Bool { true }
```

```swift
// Corner-adaptive safe margins
let guide = view.layoutGuide(for: .margins(cornerAdaptation: .horizontal))
NSLayoutConstraint.activate([
    contentView.leadingAnchor.constraint(equalTo: guide.leadingAnchor),
    contentView.trailingAnchor.constraint(equalTo: guide.trailingAnchor)
])
```

## Takeaways

- Start UIScene migration now — the legacy single-delegate lifecycle ends after iOS 26.
- Use the inspector column for settings/detail panels instead of building a custom container; it handles sidebar and popover presentation automatically across platforms.
- `prefersInterfaceOrientationLocked` lets individual view controllers opt out of rotation without the blunt `UIRequiresFullscreen` plist key.
- `splitViewControllerLayoutEnvironment` trait is the correct signal for column-layout-driven UI changes; stop reading `traitCollection.horizontalSizeClass` for split-view decisions.

---
_Source: WWDC25 Session 282 page (abstract, chapter summaries, code samples, and resource links)._
