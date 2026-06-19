# What's new in UIKit
**WWDC25 · Session 243** · [Watch](https://developer.apple.com/videos/play/wwdc2025/243/)

_Platforms:_ iOS 26, iPadOS 26, macOS Catalyst 26, tvOS 26, visionOS 26

## Overview
UIKit 2025 covers five areas: visual support for the new Liquid Glass design system, adaptivity improvements for UISplitViewController, a comprehensive new iPad menu bar API, three architectural improvements (automatic observation tracking, `updateProperties`, and the `flushUpdates` animation option), and general framework enhancements (SwiftUI scene hosting, HDR colors, Swift notifications, file URL openURL, SF Symbols 7 symbol transitions).

The session also formalizes the deprecation timeline for the legacy UIApplication-based lifecycle — apps built with the iOS 26 SDK that haven't adopted UIScene will fail to launch in the release after iOS 26.

## Key Topics

### Liquid Glass design system
UIKit components (nav bars, tab bars, alerts, popovers, split views) automatically adopt the Liquid Glass material when the app is recompiled against iOS 26 SDK. Custom components get: **background extension view** (content surfaces under the sidebar platter), **glass material** (new UIKit background material), and **scroll edge effect** (blur as content scrolls under bars). Navigation transitions are now fluid and interruptible.

### Containers and adaptivity
`UISplitViewController` gains: **inspector column support** (third column showing detail metadata), and **column drag-to-resize** with adaptive cursor shape. `UIRequiresFullscreen` is deprecated in iPadOS 26.

### iPad menu bar API
The iPad menu bar (swipe from top edge, also available with hardware keyboard) is powered by existing `UIMenuBuilder` APIs. New in iOS 26:
- **`UIMainMenuSystem.Configuration`** — declare which system commands to include/exclude, style Find as a single Search item, supply a build handler for custom additions — set once in `applicationDidFinishLaunching`
- **`UIDeferredMenuElement.usingFocus(identifier:shouldCacheItems:)`** — focus-based deferred element that walks the responder chain to fill dynamic menu content per window/profile
- `UIKeyCommand.repeatBehavior = .nonRepeatable` — prevent key-held repeat for destructive actions
- `UIMenuBuilder` — faster performance and improved diagnostics in iOS 26
- Standard actions: `performClose` (Cmd-W), "New from clipboard", text alignment, sidebar toggle, inspector toggle
- Menus defined in storyboards are no longer supported — implement programmatically

### Architectural improvements

**Automatic observation tracking**: UIKit now integrates Swift `@Observable` at its core. Observable objects referenced in `layoutSubviews`, `viewWillLayoutSubviews`, `configurationUpdateHandler`, and (new) `updateProperties` are automatically tracked — no `setNeedsLayout` required. Back-deployable to iOS 18 via `UIObservationTrackingEnabled` Info.plist key; enabled by default on iOS 26.

**`updateProperties`**: New override point on `UIView` and `UIViewController`. Runs before `layoutSubviews` but independently — invalidate properties without triggering layout. Tracks Observables automatically. Manually trigger with `setNeedsUpdateProperties()`. Use for content population, styling, and badge updates instead of `layoutSubviews`.

**`flushUpdates` animation option**: `UIView.animate(options: .flushUpdates) { ... }` applies pending Observable-driven updates and Auto Layout changes automatically — no `layoutIfNeeded()` call needed.

### General framework enhancements

**SwiftUI scene hosting**: New **`UIHostingSceneDelegate`** protocol lets UIKit apps use SwiftUI-only scene types. Implement `static var rootScene: some Scene` on your delegate, set it as the `delegateClass` in `UISceneConfiguration`, then open scenes with `UISceneSessionActivationRequest(hostingDelegateClass:id:)`.

**HDR Color support**: `UIColor(red:green:blue:alpha:linearExposure:)` — create HDR colors relative to SDR peak white. `UIColorPickerViewController.maximumLinearExposure` — enable HDR color selection in the color picker. `UITraitHDRHeadroomUsageLimit` trait — monitor when HDR content should fall back to SDR.

**Swift notifications**: Each notification is now a typed `NotificationCenter.Message`, enabling `NotificationCenter.default.addObserver(of:for:)` with strongly typed event details (e.g., `message.animationDuration`, `message.endFrame`).

**openURL for file URLs**: `UIApplication.shared.openURL(_:)` now accepts file URLs — hand off unsupported document types to the system's default app.

**SF Symbols 7**: UIButton gains `configuration.symbolContentTransition = UISymbolContentTransition(.replace)` for automatic symbol transitions on state change. New gradient color rendering mode for symbols via `UIImage.SymbolConfiguration`.

**UIScene lifecycle deprecation**: `UIApplicationDelegate` callbacks, `UIApplicationLaunchOptionsKeys`, and all `UIWindow` initializers except `init(windowScene:)` are deprecated. Adoption of UIScene lifecycle will be required after iOS 26.

## APIs & Frameworks

### UIKit
- **`UIMainMenuSystem.Configuration`** **[NEW]** — `printingPreference`, `inspectorPreference`, `findingConfiguration.style = .search`
- `UIMainMenuSystem.shared.setBuildConfiguration(_:buildHandler:)` **[NEW]**
- **`UIDeferredMenuElement.usingFocus(identifier:shouldCacheItems:)`** **[NEW]**
- `UIViewController.provider(for:)` **[NEW]** — override to supply deferred element items
- `UIKeyCommand.repeatBehavior` **[NEW]** — `.nonRepeatable`, `.defaultRepeating`
- `UIDeferredMenuElement.Identifier` — strongly typed identifier **[NEW]**
- **`UIView.updateProperties()`** **[NEW]** — override point, runs before `layoutSubviews`
- **`UIView.setNeedsUpdateProperties()`** **[NEW]**
- **`UIViewController.updateProperties()`** **[NEW]**
- **`UIViewController.setNeedsUpdateProperties()`** **[NEW]**
- **`UIView.animate(options: .flushUpdates)`** **[NEW]** — `.flushUpdates` animation option
- `UIView.AnimationOptions.flushUpdates` **[NEW]**
- **`UIHostingSceneDelegate`** protocol **[NEW]** — `static var rootScene: some Scene`
- `UISceneSessionActivationRequest(hostingDelegateClass:id:)` **[NEW]**
- **`UIColor(red:green:blue:alpha:linearExposure:)`** **[NEW]** — HDR color
- `UIColorPickerViewController.maximumLinearExposure` **[NEW]**
- **`UITraitHDRHeadroomUsageLimit`** **[NEW]** — trait for headroom monitoring
- Swift Notifications: `NotificationCenter.Message` per notification type **[NEW]**; `NotificationCenter.default.addObserver(of:for:_:)` **[NEW]**
- `UIApplication.shared.openURL(_:)` — now accepts file URLs **[NEW behavior]**
- `UIButton.Configuration.symbolContentTransition` **[NEW]** — `UISymbolContentTransition(.replace)`
- SF Symbols gradient: `UIImage.SymbolConfiguration` gradient rendering **[NEW]**
- `UISplitViewController` inspector column support **[NEW]**
- `UISplitViewController` column drag-to-resize **[NEW]**
- **Automatic observation tracking** in UIKit update methods — `UIObservationTrackingEnabled` Info.plist key (iOS 18 back-deploy) **[NEW]**
- `UIRequiresFullscreen` — **deprecated** in iPadOS 26
- `UIApplicationDelegate` lifecycle callbacks — **deprecated**

## Code Highlights

```swift
// Main menu configuration
var config = UIMainMenuSystem.Configuration()
config.printingPreference = .included
config.inspectorPreference = .removed
config.findingConfiguration.style = .search
UIMainMenuSystem.shared.setBuildConfiguration(config) { builder in
    builder.insertElements([...], afterCommand: #selector(copy(_:)))
}

// updateProperties with @Observable
@Observable class BadgeModel { var badgeCount: Int? }
override func updateProperties() {
    super.updateProperties()
    folderButton.badge = model.badgeCount.map { .count($0) }
}

// flushUpdates for automatic animation
UIView.animate(options: .flushUpdates) {
    model.badgeColor = .red          // Observable-driven update animates automatically
    topSpacingConstraint.constant = 20  // Auto Layout constraint also animates
}

// UIHostingSceneDelegate
class ZenGardenSceneDelegate: UIResponder, UIHostingSceneDelegate {
    static var rootScene: some Scene {
        WindowGroup(id: "zengarden") { ZenGardenView() }
        #if os(visionOS)
        ImmersiveSpace(id: "zengardenspace") { ZenGardenSpace() }
        #endif
    }
}

// Swift notifications — typed keyboard event
NotificationCenter.default.addObserver(of: UIScreen.self, for: .keyboardWillShow) { message in
    UIView.animate(withDuration: message.animationDuration, options: .flushUpdates) {
        bottomConstraint.constant = view.bounds.maxY - message.endFrame.minY
    }
}

// Symbol content transition
var config = UIButton.Configuration.plain()
config.symbolContentTransition = UISymbolContentTransition(.replace)
```

## Takeaways
- Compile against the iOS 26 SDK to pick up the Liquid Glass design system automatically; add a `UIBackgroundExtensionView` behind your sidebar content for the most polished appearance.
- Adopt `updateProperties` for content and styling logic currently in `layoutSubviews` — it runs independently, tracks Observables automatically, and avoids redundant layout passes.
- Migrate to the UIScene lifecycle now — UIApplication-based lifecycle apps will cease to launch in the next release after iOS 26; use "Migrating to the UIKit scene-based life cycle" tech note as a guide.
- Implement iPad menu bar support using `UIMainMenuSystem.Configuration` and `UIDeferredMenuElement.usingFocus` for context-sensitive menus — the menu bar is now accessible without a hardware keyboard.

---
_Source: WWDC25 Session 243 page (abstract, chapter summaries, code samples, and resource links)._
