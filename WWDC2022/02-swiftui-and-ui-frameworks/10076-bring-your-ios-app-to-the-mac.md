# Bring Your iOS App to the Mac
**WWDC22 · Session 10076** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10076/)

_Platforms:_ macOS Ventura 13, iOS 16, iPadOS 16

## Overview
This session surveys the full spectrum of options for getting an iOS app onto macOS: running as a native iOS app on M1 Macs (with two new Info.plist keys for screen sizing and fullscreen behavior), adding Touch Alternatives for games, and taking the full Mac Catalyst route. It then focuses on what developers get for free by adopting iPadOS 16 desktop-class APIs in a Mac Catalyst app, and covers new Mac Catalyst-specific APIs for window geometry, window control customization, custom views in NSToolbar, and refined navigation bar behavior.

The desktop-class iPad enhancements (centered toolbar items, document menus, search bar in toolbar) translate automatically to native macOS representations in Mac Catalyst apps. New APIs in macOS Ventura add programmatic window sizing/repositioning, per-window traffic light button state control, `NSUIViewToolbarItem` for embedding UIViews in NSToolbar, and popovers anchored to toolbar items.

## Key Topics

### iOS Apps on M1 Mac (No Code Changes)
- `UISupportsTrueScreenSizeOnMac` (Info.plist Bool) — app receives true screen dimensions and pixel density instead of a compatible iPad size
- `UILaunchToFullScreenByDefaultOnMac` (Info.plist Bool) — launches directly to fullscreen
- Both keys are safe to add; ignored on iOS and macOS < 12.1
- Touch Alternatives: `com.apple.uikit.inputalternatives.plist` — automatically converts keyboard/mouse/trackpad input into iOS gestures; configure `defaultEnablement` and `requiredOnboarding` keys

### Mac Catalyst Automatic Conversions (with iPadOS 16 APIs)
- `UINavigationBar` → `NSToolbar` automatically (with Mac idiom opt-in)
- Center bar button items → `NSToolbarItem`s
- Document window title and file proxy icon displayed automatically
- Navigation controller back button and navigation controls appear in toolbar
- New File menu items: Duplicate, Move, Rename, Export As (requires responder chain overrides on `UIResponder`)
- Remove unwanted File menu items via `UIMenuBuilder` using `UIMenuIdentifier.document`
- Search bar → toolbar search button that expands into `UISearchTextField` hosted in `NSToolbarItem`
- Search suggestions menu and scope bar → native AppKit controls

### Window Geometry and Controls (New Mac Catalyst APIs)
- `UIWindowScene.requestGeometryUpdate(_:errorHandler:)` — programmatically resize and reposition window **[NEW]**
- `UIWindowScene.MacGeometryPreferences(systemFrame:)` — specify exact frame in AppKit points **[NEW]**
- `UIWindowScene.effectiveGeometry` — current frame (read-only; `CGRectNull` before creation) **[NEW]**
- `UIWindowScene.windowingBehaviors.closable: Bool` — enable/disable red close button **[NEW]**
- `UIWindowScene.windowingBehaviors.miniaturizable: Bool` — enable/disable yellow minimize button **[NEW]**
- `UIWindowScene.sizeRestrictions?.allowsFullScreen: Bool` — enable/disable fullscreen **[NEW]**
- `UIWindowScene.isFullScreen: Bool` — read current fullscreen state **[NEW]**
- Coordinate space origin: upper-left of main display (the one showing the menu bar)
- Scale note: `systemFrame` is in AppKit points; iPad idiom apps differ by 77% scale factor

### Toolbar Customization (New Mac Catalyst APIs)
- `NSUIViewToolbarItem` — new `NSToolbarItem` subclass that wraps a `UIView` **[NEW]**
- Used with `NSToolbarDelegate.toolbarItemForItemIdentifier`; each customization mode requires unique `UIView` instances
- `UIBarButtonItem.customView` → automatically wrapped in toolbar if using desktop-class iPad API
- `UIPopoverPresentationController.sourceItem: UIBarButtonItem?` — present popovers anchored to toolbar items
- `UINavigationBar.preferredBehavioralStyle` — `.automatic` (default), `.mac` (force translation), `.pad` (opt out of translation) **[NEW]**

## APIs & Frameworks

**UIKit — macOS Native iOS App (Info.plist keys)** **[NEW]**
- `UISupportsTrueScreenSizeOnMac: Bool` **[NEW]**
- `UILaunchToFullScreenByDefaultOnMac: Bool` **[NEW]**
- `com.apple.uikit.inputalternatives.plist` — Touch Alternatives configuration

**UIKit — Mac Catalyst Window Management** **[NEW]**
- `UIWindowScene.MacGeometryPreferences` **[NEW]**
  - `init(systemFrame: CGRect)`
- `UIWindowScene.requestGeometryUpdate(_ preferences:errorHandler:)` **[NEW]**
- `UIWindowScene.effectiveGeometry: UIWindowScene.GeometryPreferences.Mac` **[NEW]**
- `UIWindowScene.windowingBehaviors: UISceneWindowingBehaviors?` **[NEW]**
  - `.closable: Bool`
  - `.miniaturizable: Bool`
- `UIWindowScene.sizeRestrictions?.allowsFullScreen: Bool` **[NEW]**
- `UIWindowScene.isFullScreen: Bool` **[NEW]**

**UIKit — Toolbar** **[NEW]**
- `NSUIViewToolbarItem` (AppKit/UIKit bridge class) **[NEW]**
  - `init(itemIdentifier:uiView:)` or similar
- `UINavigationBar.preferredBehavioralStyle: UIBehavioralStyle` **[NEW]**
  - `.automatic`, `.mac`, `.pad`
- `UIBarButtonItem.customView` — existing property; auto-wrapped into `NSToolbarItem` when using Mac idiom

**UIKit — Document Menu Items**
- `UIResponder.duplicate(_:)`, `.move(_:)`, `.rename(_:)`, `.exportToService(_:)` — override to enable File menu items
- `UIMenuIdentifier.document` — identifier for the document submenu

## Code Highlights

Window geometry at scene creation:
```swift
func scene(_ scene: UIScene, willConnectTo session: UISceneSession, ...) {
    guard let windowScene = scene as? UIWindowScene else { return }
    var frame = windowScene.effectiveGeometry.systemFrame
    frame.size = CGSize(width: 400, height: 600)
    let prefs = UIWindowScene.MacGeometryPreferences(systemFrame: frame)
    windowScene.requestGeometryUpdate(prefs) { error in
        // Handle rejection
    }
    windowScene.windowingBehaviors?.miniaturizable = false
    windowScene.sizeRestrictions?.allowsFullScreen = false
}
```

Repositioning a window at any time:
```swift
var frame = scene.effectiveGeometry.systemFrame
frame.origin = CGPoint(x: 100, y: 100)
scene.requestGeometryUpdate(UIWindowScene.MacGeometryPreferences(systemFrame: frame))
```

Custom UIView in NSToolbar:
```swift
// In NSToolbarDelegate:
func toolbar(_ toolbar: NSToolbar, itemForItemIdentifier ...) -> NSToolbarItem? {
    return NSUIViewToolbarItem(itemIdentifier: identifier, uiView: MyCustomView())
}
```

## Takeaways
- Two Info.plist keys (`UISupportsTrueScreenSizeOnMac`, `UILaunchToFullScreenByDefaultOnMac`) provide significant native-feel improvements for iOS apps on M1 with zero code changes.
- Adopting desktop-class iPad APIs (centered toolbar items, document menus, search) gives Mac Catalyst apps native macOS representations for free.
- New `UIWindowScene` geometry and windowing behavior APIs enable fine-grained control over window size, position, traffic light buttons, and fullscreen on a per-scene basis.
- `NSUIViewToolbarItem` closes the gap for apps that manage their own `NSToolbar` and want to embed SwiftUI or UIKit views directly in the toolbar.

---
_Source: WWDC22 Session 10076 page (abstract, chapter summaries, code samples, and resource links)._
