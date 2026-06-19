# Modernize Your UIKit App
**WWDC26 · Session 278** · [Watch](https://developer.apple.com/videos/play/wwdc2026/278/)

_Platforms:_ iOS, iPadOS (primary), macOS (Catalyst / iPhone Mirroring)

## Overview
With iOS 27, iPhone apps are now fully resizable — they can be freely sized in iPhone Mirroring on Mac and run at arbitrary sizes on iPad. This session identifies the four legacy API patterns that break in resizable environments and provides direct replacements for each, then covers new APIs for tab bars, navigation bars, and menus, and introduces an Xcode 27 agentic skill that automates much of the migration work.

A key architectural requirement: the UIScene lifecycle is now mandatory when building with the iOS 27 SDK — apps still using the UIApplication app-delegate lifecycle will no longer launch. This session provides the roadmap for each modernization step, supplemented by a new agentic coding skill that can automatically convert many of these patterns.

## Key Topics

### App Adaptivity (0:34)
iPhone apps running in iPhone Mirroring or on iPad can now be freely resized to any window size. The four legacy patterns to audit and fix:
1. **App lifecycle** — migrate from UIApplicationDelegate to UISceneDelegate (now required)
2. **Main screen references** — `UIScreen.main` refers to the device screen, not the scene's screen; replace with `window?.windowScene?.screen`
3. **User interface idiom** — `UIDevice.current.userInterfaceIdiom` is no longer meaningful for layout; use size classes
4. **Interface orientation** — `UIInterfaceOrientation` is ignored in resizable environments; replace with size classes

### Legacy API Replacements (2:51 – 7:55)
- Replace `UIScreen.main.scale` with `traitCollection.displayScale`; register for `UITraitDisplayScale` changes via `registerForTraitChanges`
- Replace screen-based available-space calculations with `windowScene.effectiveGeometry.coordinateSpace.bounds` or simply `view.bounds.size`
- **[NEW]** `UIWindowSceneDelegate.windowScene(_:didUpdateEffectiveGeometry:)` fires on resize
- `UIRequiresFullScreen` Info.plist key is now honored on iPhone in resizable environments — use this for games that need a fixed orientation
- **[NEW]** `UIView` conforms to CoreMotion and CoreLocation `Body` protocols in iOS 27, keeping sensor data in the correct coordinate space regardless of orientation: `motionManager.deviceMotionBody = view` / `locationManager.headingBody = view`

### Tab Bars and Sidebars (9:18)
- **[NEW]** `tabBarController.sidebar.preferredPlacement = .sidebar` — opt iPhone apps into the sidebar layout when space is available
- **[NEW]** `tabBarController.sidebar.isAvailable` — check if the system will show the sidebar
- **[NEW]** `tabBarController.prominentTabIdentifier` — pin a tab so it stays visible when the tab bar collapses during scroll (UIKit equivalent of `Tab(role: .prominent)` in SwiftUI)

### Navigation Bars (10:52)
- **[NEW]** `navigationItem.barMinimizationBehavior` — control when the navigation bar auto-minimizes on scroll (`.always`, `.never`, `.automatic`)
- **[NEW]** `navigationItem.barMinimizationSafeAreaAdjustment` — whether safe area adjusts when the bar minimizes
- The `.automatic` scroll edge effect style has updated visuals in iOS 27; apps that previously overrode to `.soft` should re-evaluate

### Menus and Apple Intelligence (12:37 – 13:01)
- **[NEW]** `UIMenuElement.preferredImageVisibility` — force images to show in menu elements even in contexts (menu bar on iPadOS/macOS) where they're hidden by default
- Menus now automatically display an "Ask Siri" button when content is relevant; drag sessions can be initiated without user gesture — avoid triggering animations or presenting modal UI from `sessionWillBegin`

### Agentic Coding (14:07)
Xcode 27 includes an app-modernization coding skill that automatically converts `UIScreen.main` calls, replaces orientation checks with size class checks, and migrates to scene lifecycle. Skills can be exported with `xcrun agent skills export` for use in other coding agents.

## APIs & Frameworks

**UIKit**
- `UISceneDelegate` / `UIWindowSceneDelegate` — **required** from iOS 27 SDK
- `UIWindowScene.screen` — correct per-scene screen reference
- `UIWindowScene.effectiveGeometry` — **[NEW]** current geometry in resizable environments
- `UIWindowScene.Geometry.coordinateSpace.bounds`
- `UIWindowSceneDelegate.windowScene(_:didUpdateEffectiveGeometry:)` — **[NEW]** resize callback
- `UITraitCollection.displayScale` — replaces `UIScreen.main.scale`
- `UITraitDisplayScale` — trait for `registerForTraitChanges`
- `registerForTraitChanges(_:handler:)`
- `UIRequiresFullScreen` — Info.plist key, now honored on iPhone
- `UITabBarController.sidebar.preferredPlacement` — **[NEW]** `.sidebar`
- `UITabBarController.sidebar.isAvailable` — **[NEW]**
- `UITabBarController.prominentTabIdentifier` — **[NEW]**
- `UINavigationItem.barMinimizationBehavior` — **[NEW]** (`.always`, `.never`, `.automatic`)
- `UINavigationItem.barMinimizationSafeAreaAdjustment` — **[NEW]**
- `UIScrollEdgeAppearance` — `.automatic` style updated in iOS 27
- `UIMenuElement.preferredImageVisibility` — **[NEW]**

**CoreMotion / CoreLocation**
- `CMMotionManager.deviceMotionBody` — **[NEW]** assign a UIView to auto-rotate motion data
- `CLLocationManager.headingBody` — **[NEW]** assign a UIView to auto-rotate heading data

**Xcode / Toolchain**
- `xcrun agent skills export` — **[NEW]** export Xcode modernization skills for external agents

## Code Highlights

Replace `UIScreen.main` with scene screen:
```swift
let screen = window?.windowScene?.screen
```

Replace scale with trait-based display scale:
```swift
override func layoutSubviews() {
    super.layoutSubviews()
    let displayScale = traitCollection.displayScale
}
```

Opt into sidebar on iPhone:
```swift
tabBarController.sidebar.preferredPlacement = .sidebar
```

Navigation bar minimize on scroll:
```swift
navigationItem.barMinimizationBehavior = .always
navigationItem.barMinimizationSafeAreaAdjustment = .never
```

## Takeaways
- UIScene lifecycle is no longer optional — apps built with the iOS 27 SDK that don't use it will not launch; migrate immediately.
- Replace every `UIScreen.main` reference with the window scene's screen or `traitCollection.displayScale`; this is the most common crash cause in iPhone Mirroring.
- Use size classes (not interface orientation or user interface idiom) as the basis for all layout decisions going forward.
- Run the Xcode 27 app-modernization agent skill to automate the majority of these conversions before auditing the edge cases manually.

---
_Source: WWDC26 Session 278 page (abstract, chapter summaries, code samples, and resource links)._
