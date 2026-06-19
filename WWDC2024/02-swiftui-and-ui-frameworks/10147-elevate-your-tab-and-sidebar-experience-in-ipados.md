# Elevate Your Tab and Sidebar Experience in iPadOS
**WWDC24 · Session 10147** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10147/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, visionOS 2, tvOS 18

## Overview
iPadOS 18 redesigns the tab bar with a compact layout that floats at the top of the app alongside navigation controls, leaving more vertical space for content. The same `TabView`/`UITabBarController` API now optionally morphs into a full sidebar, bringing together two previously separate navigation paradigms. Users can customize tab order and visibility through drag and drop, with persistence handled automatically.

The session covers both SwiftUI and UIKit APIs in parallel, ensuring developers can adopt these patterns whether they use declarative or imperative UI code. Cross-platform behavior is explicitly discussed for macOS, visionOS, and tvOS.

## Key Topics

### Tab Bar and Sidebar Refresh
Recompiling with the iOS 18 SDK automatically adopts the new compact tab bar look — no code changes needed. The tab bar now shares safe area space with the navigation bar, and toolbar items overflow gracefully when space is tight.

### New Tab API (SwiftUI)
`Tab` is a new struct replacing the old `tabItem` modifier pattern. It carries a title, image, optional selection value, and content. `Tab(role: .search)` auto-configures a pinned search tab. `TabSection` groups tabs for sidebar display. Apply `.tabViewStyle(.sidebarAdaptable)` to enable sidebar mode.

### New Tab API (UIKit)
`UITab` and `UITabGroup` replace the old `viewControllers` array on `UITabBarController`. Set `tabBarController.mode = .tabSidebar` to enable the sidebar. `UISearchTab` creates a pinned search tab automatically. Groups support dynamic child updates and sidebar `sidebarActions`.

### Drag-and-Drop Destinations
Both `Tab` (SwiftUI `.dropDestination(for:action:)`) and `UITab` (delegate methods) support becoming drop destinations in the tab bar and sidebar.

### User Customization
`TabViewCustomization` (SwiftUI) or `allowsHiding`/`allowsReordering` (UIKit) let users hide or reorder tabs. `@AppStorage` persists customization automatically. `customizationBehavior(.disabled, for: .sidebar, .tabBar)` pins critical tabs. `defaultVisibility(.hidden, for: .tabBar)` hides optional tabs by default.

## APIs & Frameworks

**SwiftUI**
- `Tab(_:systemImage:value:content:)` **[NEW]**
- `Tab(role: .search) { ... }` **[NEW]**
- `TabSection(_:content:)` **[NEW]**
- `.tabViewStyle(.sidebarAdaptable)` **[NEW]**
- `.tabViewCustomization($customization)` modifier **[NEW]**
- `TabViewCustomization` **[NEW]**
  - `@AppStorage` persistence support
- `.customizationID(_:)` modifier on `Tab`/`TabSection` **[NEW]**
- `.customizationBehavior(_:for:)` modifier **[NEW]**
- `.defaultVisibility(_:for:)` modifier **[NEW]**
- `.dropDestination(for:action:)` on `Tab` **[NEW]**
- `.sectionActions { Button(...) }` on `TabSection` **[NEW]**

**UIKit**
- `UITab(title:image:identifier:viewControllerProvider:)` **[NEW]**
- `UITabGroup(title:image:identifier:children:viewControllerProvider:)` **[NEW]**
- `UISearchTab(viewControllerProvider:)` **[NEW]**
- `UITabBarController.tabs: [UITab]` **[NEW]**
- `UITabBarController.mode = .tabSidebar` **[NEW]**
- `UITab.allowsHiding: Bool` **[NEW]**
- `UITab.isHidden: Bool` **[NEW]**
- `UITab.preferredPlacement: UITab.Placement` **[NEW]** (`.default`, `.optional`, `.movable`, `.pinned`, `.fixed`, `.sidebarOnly`)
- `UITabGroup.allowsReordering: Bool` **[NEW]**
- `UITabGroup.displayOrderIdentifiers: [String]` **[NEW]**
- `UITabGroup.sidebarActions: [UIAction]` **[NEW]**
- `UITabGroup.children: [UITab]` — mutable for dynamic updates **[NEW]**
- `UITabBarControllerDelegate` new methods: **[NEW]**
  - `tabBarController(_:visibilityDidChangeFor:)`
  - `tabBarController(_:displayOrderDidChangeFor:)`
  - `tabBarController(_:tab:operationForAcceptingItemsFrom:)`
  - `tabBarController(_:tab:acceptItemsFrom:)`

## Code Highlights

```swift
// SwiftUI sidebar tab view
TabView {
    Tab("Watch Now", systemImage: "play") { WatchNowView() }
    TabSection("Collections") {
        Tab("Cinematic Shots", systemImage: "list.and.film") { ... }
    }
    Tab(role: .search) { SearchView() }
}
.tabViewStyle(.sidebarAdaptable)
.tabViewCustomization($customization)
```

```swift
// UIKit tab sidebar
tabBarController.mode = .tabSidebar
tabBarController.tabs = [
    UITab(title: "Watch Now", image: UIImage(systemName: "play"), identifier: "Tabs.watchNow") { _ in WatchNowViewController() },
    UITabGroup(title: "Collections", ..., children: collectionsTabs()) { _ in ... },
    UISearchTab { _ in SearchViewController() }
]
```

## Takeaways
- Recompile with the iOS 18 SDK to get the new tab bar layout automatically; then add `Tab`/`UITab` for richer structure.
- Use `.sidebarAdaptable` / `.tabSidebar` to give iPad users a collapsible sidebar without rebuilding navigation.
- Add `TabViewCustomization` (SwiftUI) or `allowsHiding`/`allowsReordering` (UIKit) so users can personalize the tab bar.
- The same `Tab`/`UITab` API adapts appropriately on macOS (sidebar), visionOS 2 (ornament tab bar), and tvOS 18 (collapsible sidebar).

---
_Source: WWDC24 Session 10147 page (abstract, chapter summaries, code samples, and resource links)._
