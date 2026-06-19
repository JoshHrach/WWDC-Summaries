# Taking iPad Apps for Mac to the Next Level
**WWDC19 · Session 235** · [Watch](https://developer.apple.com/videos/play/wwdc2019/235/)

_Platforms:_ macOS Catalina 10.15, iOS 13, iPadOS 13

## Overview
Mac Catalyst (then known as "iPad Apps for Mac") lets developers bring iPad apps to macOS by checking a single checkbox in Xcode. This session — the follow-up to "Introducing iPad Apps for Mac" — goes beyond the basics to show developers how to make their apps feel natively at home on Mac. Topics include UIKit APIs newly available on macOS (sidebars, toolbars, context menus, hover), building an NSToolbar via UIWindowScene's titlebar, customizing the menu bar with UIMenuBuilder, the macOS application lifecycle for UIKit apps, and distribution specifics.

## Key Topics

### Better iPad Apps Are Better Mac Apps
- Auto Layout and Dynamic Type are even more important on Mac because windows can be resized to 35-inch-equivalent screen sizes at the Mac's 77% scaling factor.
- Implement keyboard shortcuts (`UIKeyCommand`) — Mac users always have a keyboard; they work on iPadOS too via the discoverability HUD.
- Adopt drag and drop (`UIDragInteraction`, `UIDropInteraction`) — Mac users expect it.
- Migrate off deprecated APIs: use `WKWebView` (not `UIWebView`), use Metal (not OpenGL ES).
- Support Multi-Window (iOS 13) — every `UIWindowScene` becomes a Mac window.
- Adopt Dark Mode on iPad — you get it on Mac automatically.

### Mac-Specific UIKit Enhancements

**Sidebar style for UISplitViewController:**
- `UISplitViewController.primaryBackgroundStyle = .sidebar` — gives the primary column translucent sidebar appearance.
- Embedded table views in sidebar automatically adopt source-list (sidebar icon size preference respected).

**NSToolbar via UIWindowScene.titlebar** **[NEW for UIKit apps]**
- `UIWindowScene.titlebar` — exposes the `UITitlebar` object on Mac. **[NEW]**
- Set `titlebar.toolbar` to an `NSToolbar` instance.
- `titlebar.titleVisibility = .hidden` — hides the title (useful when a navigation item already provides context).
- `NSToolbar.allowsUserCustomization = true` — enables user-customizable toolbar.
- Toolbar items are configured via `NSToolbarDelegate`.

**Context menus:**
- `UIContextMenuInteraction` — new cross-platform API (iOS 13 + Mac). Uses `UIMenu` and `UIAction` for dynamic context menus shown on right-click/control-click on Mac. **[NEW]**
- `UIAction` — block-based command; appropriate for context menus. **[NEW]**

**Hover gesture:**
- `UIHoverGestureRecognizer` — new gesture recognizer that tracks mouse position without clicking; does not exist on iPad but works seamlessly on Mac. **[NEW]**

**Touch Bar:**
- `NSTouchBar` exposed to UIKit apps via new `UIResponder` and `UIViewController` APIs. **[NEW for UIKit]**

### Menu Bar Customization **[NEW]**
- `UICommand` — new superclass of `UIKeyCommand` for representing commands. **[NEW]**
- `UIMenu` — represents a menu containing commands or sub-menus. **[NEW]**
- `UIMenuBuilder` — passed to `buildMenu(with:)` on `UIApplicationDelegate` to compose the global menu bar. **[NEW]**
- `UIMenuBuilder.remove(menu:)` — removes an entire system-provided menu (e.g., Format menu).
- `UIMenuBuilder.insertChild(_:atStartOfMenu:)` / `insertChild(_:atEndOfMenu:)` — adds custom commands.
- `UIMenu.Options.displayInline` — displays menu children inline (flat) rather than as a hierarchical submenu.
- Key commands placed in menus also appear in iPadOS discoverability HUD — no `#if os()` conditional needed.
- Override `buildMenu(with:)` in `UIApplicationDelegate`; guard on `builder.system == .main` to target the main menu bar.

### Application Lifecycle on macOS
- Same delegate calls and notifications as iOS: `didFinishLaunching`, `didBecomeActive`, `willResignActive`, `didEnterBackground`, `willTerminate`.
- **Key difference:** On Mac, UIKit apps remain **foreground active almost all the time** — switching apps, occluding windows, or losing the menu bar does NOT trigger willResignActive/didEnterBackground.
- System handles throttling via **AppNap** automatically (based on visibility, audio, etc.) — no need to reduce frame rate or stop timers manually on deactivation.
- Apps enter background only during **termination** or **background launches** (not during app switching).
- Background audio plist entitlements are **ignored on Mac** — audio pauses on background, but background audio isn't needed since the app stays foreground active.
- Memory: no enforced memory limits on Mac (Mac memory model); no OOM kills, but leaks accumulate longer — fix them with Allocations instrument.
- Suspension: iPad apps on Mac are suspended only during background launches; during termination, they always exit (never just suspend).

### Supported Background Launch APIs
- `URLSession` background downloads
- Silent remote notifications (`content-available`)
- Notification actions without `foreground` option
- `BGProcessingTask` and `BGAppRefreshTask` (BackgroundTasks framework)

### Distribution
- Mac app gets a new bundle ID prefixed with the "maccatalyst." (previously "uikitformac.") prefix — registered automatically by Xcode.
- Uses a **unified Apple developer certificate** (one cert for iOS + Mac). **[NEW]**
- Automatic signing in Xcode handles Mac provisioning profiles.
- Hardened Runtime and App Sandbox entitlements added automatically.
- Shared entitlements file between iOS and Mac; Xcode migrates camera/iCloud container entitlements.
- Keychain Sharing capability must be added manually if using Keychain.
- Push notifications: continue sending to the iOS app identifier — both iOS and Mac receive them.
- **App Thinning is not used** on Mac; the package contains all resources.
- Receipt validation must be implemented — Mac apps can be copied and moved.
- Separate App Store Connect record required for Mac app; use the uikitformac bundle ID.
- Increment build number on every Mac upload (macOS uses build numbers for release ordering).
- In-App Purchases must be recreated in App Store Connect for the Mac app record.

## APIs & Frameworks

### UIKit — Mac Catalyst Additions **[NEW]**
- `UIWindowScene.titlebar` → `UITitlebar` — access to title bar and toolbar on Mac **[NEW]**
- `UITitlebar.toolbar` — attach an `NSToolbar` **[NEW]**
- `UITitlebar.titleVisibility` — show/hide window title **[NEW]**
- `UISplitViewController.primaryBackgroundStyle` — `.sidebar` for Mac sidebar appearance **[NEW]**
- `UIHoverGestureRecognizer` — mouse hover tracking gesture **[NEW]**
- `UIContextMenuInteraction` — right-click context menus **[NEW]**
- `UIMenu` — menu containing commands **[NEW]**
- `UIAction` — block-based command **[NEW]**
- `UICommand` — responder-chain-based command **[NEW]**
- `UIMenuBuilder` — compose/modify the app's menu bar **[NEW]**
- `UIApplicationDelegate.buildMenu(with:)` — override to customize menu bar **[NEW]**
- `NSTouchBar` — Touch Bar, exposed via UIResponder/UIViewController **[NEW for UIKit]**

### AppKit APIs Available to UIKit Apps
- `NSToolbar` — Mac toolbar (attached via `UITitlebar`)
- `NSToolbarDelegate` — configure toolbar items

## Code Highlights

Set sidebar background style on UISplitViewController:
```swift
// Conditionalized for Mac only
#if targetEnvironment(macCatalyst)
splitViewController.primaryBackgroundStyle = .sidebar
#endif
```

Attach NSToolbar to window via UIWindowScene:
```swift
// In UISceneDelegate.scene(_:willConnectTo:options:)
if let titlebar = windowScene.titlebar {
    let toolbar = NSToolbar(identifier: "MainToolbar")
    toolbar.delegate = toolbarDelegate
    toolbar.allowsUserCustomization = true
    titlebar.toolbar = toolbar
    titlebar.titleVisibility = .hidden
}
```

Customize the global menu bar:
```swift
// In UIApplicationDelegate
override func buildMenu(with builder: UIMenuBuilder) {
    guard builder.system == .main else { return }
    
    // Remove Format menu
    builder.remove(menu: .format)
    
    // Add custom commands to File menu
    let newRecipeCommand = UIKeyCommand(
        title: "New Recipe",
        action: #selector(createRecipe),
        input: "f",
        modifierFlags: [.command, .alternate]
    )
    let favCommand = UICommand(title: "Make Favorite", action: #selector(toggleFavorite))
    let menu = UIMenu(title: "", options: .displayInline, children: [newRecipeCommand, favCommand])
    builder.insertChild(menu, atStartOfMenu: .file)
}
```

UIContextMenuInteraction on a view:
```swift
let interaction = UIContextMenuInteraction(delegate: self)
recipeImageView.addInteraction(interaction)

// In UIContextMenuInteractionDelegate:
func contextMenuInteraction(_ interaction: UIContextMenuInteraction,
    configurationForMenuAtLocation location: CGPoint) -> UIContextMenuConfiguration? {
    return UIContextMenuConfiguration(identifier: nil, previewProvider: nil) { _ in
        let favorite = UIAction(title: "Add to Favorites") { _ in self.toggleFavorite() }
        return UIMenu(title: "", children: [favorite])
    }
}
```

## Takeaways
- Adopting best-practice iPad APIs (Auto Layout, Dynamic Type, UIKeyCommand, drag-and-drop, Dark Mode) translates directly into a better Mac app with zero extra Mac-specific code.
- The new `UIMenuBuilder`/`UIMenu`/`UICommand` system lets you compose a platform-native Mac menu bar entirely in UIKit, and `UIKeyCommand` shortcuts automatically surface in iPadOS's discoverability HUD.
- Mac UIKit apps stay **foreground active** almost always — do not expect lifecycle state changes to fire on app switch or window occlusion; rely on AppNap for throttling.
- Mac distribution requires a separate App Store Connect app record, uses a synthesized bundle ID, and requires receipt validation since apps can be copied freely.

---
_Source: WWDC19 Session 235 page (abstract, transcript, and resource links)._
