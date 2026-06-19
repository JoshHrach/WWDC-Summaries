# What's New in Mac Catalyst
**WWDC20 · Session 10143** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10143/)

_Platforms:_ macOS Big Sur 11, iPadOS 14

## Overview
Mac Catalyst received major upgrades in macOS Big Sur, dramatically expanding framework availability, improving application lifecycle behavior, and introducing "Optimized for Mac" mode. Dozens of iOS frameworks that were previously unavailable on macOS — including ARKit — are now accessible to Mac Catalyst apps, enabling developers to port iOS applications with less conditional compilation.

The new "Optimized for Mac" mode scales UIs at 100% rather than 77%, producing crisper text and graphics, and unlocks additional Mac-native controls like checkboxes. Application lifecycle semantics were aligned more closely with iOS, so existing background handling logic behaves predictably on macOS.

Universal Purchase is now the default for all new Mac Catalyst apps, allowing users who purchase on iOS to automatically access the Mac version without an additional purchase.

## Key Topics

### Expanded Framework Availability
Dozens of previously unavailable iOS frameworks are now accessible to Mac Catalyst apps targeting macOS Big Sur. ARKit, for example, can now be linked and used with runtime availability checks, matching how it behaves on iOS devices without AR support.

### Optimized for Mac Mode
A new Mac Catalyst rendering mode that changes default scaling from 77% to 100%. Controls adapt their metrics and behaviors to match Mac conventions. New APIs expose Mac-specific controls like checkboxes.

### SwiftUI Integration
SwiftUI code in a UIKit iPad app works as-is in Mac Catalyst. SwiftUI `commands` integrate with the Mac main menu, and the new `toolbar` support places items correctly in Mac toolbars using semantic placement.

### Application Lifecycle Changes
Scene lifecycle transitions were expanded: a scene moves to background when its window is minimized, the containing Space moves out of view, or the app is hidden. The overall app is considered foreground when at least one window is foreground or the app holds the menu bar. Catalyst apps do not get process suspension when backgrounded.

### Extension Lifecycle
Mac Catalyst extensions now use iOS-style memory limits and memory pressure behavior. Extensions are suspended (not terminated) when not in use. Photo editing extensions are now supported.

### WidgetKit Support
WidgetKit widgets built for iPadOS automatically work on macOS when the app opts in to Mac Catalyst.

### Universal Purchase
All new Mac Catalyst apps default to Universal Purchase. One purchase gives users access on all platforms. Apps use a shared iOS bundle identifier; opting out requires unchecking "Use iOS Bundle Identifier" in project settings.

### New macOS Big Sur UI Adoption
- **Toolbar Styles** — `UITitlebarToolbarStyle` options available per window via `UITitlebar`
- **Separator Identifiers** — Position toolbar items using sidebar and flexible-space separators in `toolbarDefaultItemIdentifiers`
- **Accent Colors** — Symbolic color in Assets catalog becomes the app's global tint color
- **Sidebar Drag Reordering** — `UICollectionView` in sidebars gains automatic drag reordering with no code changes

## APIs & Frameworks

### UIKit
- `UISceneActivationRequestOptions.CollectionJoinBehavior` **[NEW]** — controls whether a new scene opens as a tab or separate window
- `UIColorPickerViewController` **[NEW]** — maps to system AppKit color picker on Mac
- `UIColorWell` **[NEW]** — color well control
- `UIDatePicker` — now uses AppKit inline date picker on Mac **[UPDATED]**
- `UIButton` pull-down menus — now render as native Mac pull-down menus **[UPDATED]**
- Sheet presentations — now appear in separate resizable `NSWindow` **[UPDATED]**
- Popover presentations — now appear in `NSPopover` with its own window **[UPDATED]**
- `UISplitViewController` — three-column layout with sidebar support **[UPDATED]**
- `selectionFollowsFocus` — controls keyboard focus selection behavior in table/collection views
- `pressesBegan(_:with:)`, `pressesEnded(_:with:)` — physical keyboard event handling

### AppKit / Mac Catalyst Bridging
- `NSCursor` **[EXPOSED TO CATALYST]** — hide cursor, change cursor image
- `UITitlebarToolbarStyle` **[NEW]** — toolbar style per window (unified, expanded, preference, etc.)
- `UITitlebar` — per-window toolbar style configuration **[NEW]**
- Toolbar separator identifiers (sidebar separator, flexible space) **[NEW]**
- Accent color via Assets catalog symbolic color **[NEW]**

### Focus Engine (from tvOS)
- `focusGroupIdentifier` — groups related views for keyboard navigation **[NEW TO CATALYST]**
- Focus change callbacks for UI updates

### SwiftUI
- `commands` modifier — integrates with Mac main menu **[NEW]**
- `toolbar` modifier with semantic placement **[NEW]**

### WidgetKit
- Mac Catalyst widget opt-in **[NEW]** — iPad WidgetKit widgets run on macOS

### Universal Purchase
- Shared iOS Bundle Identifier — enables cross-platform purchase **[NEW DEFAULT]**

### Other Frameworks
- ARKit — now available in macOS SDK for Catalyst (graceful runtime fallback) **[NEW]**
- Photo Editing Extensions — now supported on Mac **[NEW]**

## Code Highlights

Toolbar separator identifiers example (similar to Messages app):
```swift
override func toolbarDefaultItemIdentifiers(_ toolbar: NSToolbar) -> [NSToolbarItem.Identifier] {
    return [
        .compositeButton,
        .sidebarTrackingSeparator,   // jumps out of sidebar
        .toField,
        .flexibleSpace,              // right-justifies remaining items
        .infoButton
    ]
}
```

Setting an accent color: define a symbolic color named `AccentColor` in the Assets catalog; UIKit uses it as the default tint color app-wide.

## Takeaways

- "Optimized for Mac" mode delivers pixel-perfect Mac rendering at 100% scale and should be adopted for the most refined Mac Catalyst experience.
- Universal Purchase is now on by default — new Catalyst apps share a bundle ID with iOS, giving users seamless cross-platform access.
- ARKit and dozens of other previously unavailable iOS frameworks can now be linked in Mac Catalyst targets with runtime availability checks, eliminating many `#if targetEnvironment(macCatalyst)` guards.
- Application lifecycle on macOS Big Sur is more iOS-like: scenes background on minimize/hide, making existing iOS lifecycle handlers behave correctly on Mac.

---
_Source: WWDC20 Session 10143 page (abstract, transcript, and resource links)._
