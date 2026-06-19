# Build an AppKit App with the New Design
**WWDC25 · Session 310** · [Watch](https://developer.apple.com/videos/play/wwdc2025/310/)

_Platforms:_ macOS Tahoe 26

## Overview
This session walks through updating an AppKit Mac app to adopt macOS Tahoe's new design system, mirroring the SwiftUI session but from the AppKit perspective. The presenter shows how Liquid Glass materializes through new view classes (`NSGlassEffectView`, `NSGlassEffectContainerView`), enhanced toolbar item APIs (prominence, tint, badges), updated split view safe-area behavior, a new background extension view, compact control size metrics, new border shapes, and an extra-large control size — all enabling Mac apps to feel cohesive with the new system aesthetic.

## Key Topics

### Liquid Glass Views
`NSGlassEffectView` is the AppKit primitive for Liquid Glass. It exposes a `contentView`, `cornerRadius`, and `tintColor` for customization. For groups of glass elements that should share a unified background, `NSGlassEffectContainerView` acts as a grouping container with a `spacing` property and `contentView`.

### Toolbar Updates
Toolbar items gain new styling capabilities:
- Setting `NSToolbarItem.isBordered = false` removes the default glass capsule from an item.
- `NSToolbarItem.style = .prominent` renders the item at a larger, highlighted size.
- `NSToolbarItem.backgroundTintColor` applies a tint to the item's glass background.
- `NSItemBadge` adds a count, text string, or dot indicator to any toolbar item.

### Split View and Safe Areas
`NSSplitViewItem.automaticallyAdjustsSafeAreaInsets = true` allows the split view's sidebar to correctly inset content when the sidebar expands or collapses. The new `NSView.LayoutRegion` type and `layoutGuide(for: .safeArea(cornerAdaptation: .horizontal))` give per-region safe-area layout guides.

### Background Extension and Accessories
`NSBackgroundExtensionView` extends a view's material behind the toolbar or sidebar — equivalent to SwiftUI's `.backgroundExtensionEffect`. `NSSplitViewItemAccessoryViewController` attaches an accessory below a split view item, similar to SwiftUI's bottom tab accessory.

### Controls
- `prefersCompactControlSizeMetrics` on `NSView` shrinks controls to a compact style.
- `borderShape` property on `NSButton`, `NSPopUpButton`, and `NSSegmentedControl` controls the shape of the control border.
- `.glass` bezel style for `NSButton` applies Liquid Glass styling.
- `tintProminence` property (`.none`, `.secondary`, `.primary`, `.automatic`) controls accent color intensity on controls.
- Extra-large control size is now available via `NSControl.ControlSize.extraLarge` (or equivalent size constant).
- `NSScrollView` gains a configurable scroll edge effect style (soft vs. hard fade).

## APIs & Frameworks

**AppKit (macOS Tahoe 26)**
- **[NEW]** `NSGlassEffectView` — Liquid Glass primitive; `contentView`, `cornerRadius`, `tintColor`
- **[NEW]** `NSGlassEffectContainerView` — grouped glass container; `spacing`, `contentView`
- **[NEW]** `NSToolbarItem.style = .prominent` — enlarged prominent toolbar item
- **[NEW]** `NSToolbarItem.backgroundTintColor` — tint the glass background of a toolbar item
- `NSToolbarItem.isBordered = false` — remove glass capsule from toolbar item
- **[NEW]** `NSItemBadge` — badge on toolbar items; `.count(_:)`, `.text(_:)`, `.indicator`
- `NSSplitViewItem.automaticallyAdjustsSafeAreaInsets` — sidebar-driven safe area adjustment
- **[NEW]** `NSView.LayoutRegion` — typed layout region identifier
- **[NEW]** `NSView.layoutGuide(for: .safeArea(cornerAdaptation: .horizontal))` — region-specific safe area layout guide
- **[NEW]** `NSBackgroundExtensionView` — extends view material behind toolbar/sidebar
- **[NEW]** `NSSplitViewItemAccessoryViewController` — persistent accessory below a split view item
- **[NEW]** `prefersCompactControlSizeMetrics` on `NSView` — compact control size mode
- **[NEW]** `borderShape` property on `NSButton`, `NSPopUpButton`, `NSSegmentedControl`
- **[NEW]** `.glass` bezel style for `NSButton`
- **[NEW]** `tintProminence` property — `.none`, `.secondary`, `.primary`, `.automatic`
- Extra-large control size — `NSControl.ControlSize` addition
- `NSScrollView` scroll edge effect style — soft vs. hard edge treatment

## Code Highlights
Create a glass toolbar item with badge:
```swift
let glassView = NSGlassEffectView()
glassView.cornerRadius = 8
glassView.tintColor = .systemBlue

let badge = NSItemBadge.count(3)
toolbarItem.badge = badge
toolbarItem.backgroundTintColor = .systemBlue
toolbarItem.style = .prominent
```

Group glass elements in a container:
```swift
let container = NSGlassEffectContainerView()
container.spacing = 8
container.contentView.addSubview(glassView1)
container.contentView.addSubview(glassView2)
```

Apply glass button style:
```swift
let button = NSButton()
button.bezelStyle = .glass
button.tintProminence = .primary
button.borderShape = .roundedRect
```

## Takeaways
- Use `NSGlassEffectView` for custom Liquid Glass surfaces and `NSGlassEffectContainerView` when multiple glass elements should share a unified backdrop.
- Enhance toolbar items with `.prominent` style, `backgroundTintColor`, and `NSItemBadge` to match the expressiveness of iOS 26 toolbar components.
- Adopt `automaticallyAdjustsSafeAreaInsets` and `NSBackgroundExtensionView` to ensure content correctly adapts to Tahoe's expanded chrome areas.
- Use `tintProminence` and `.glass` bezel style to give buttons the right level of visual weight in the new design system.

---
_Source: WWDC25 Session 310 page (abstract, chapter summaries, code samples, and resource links)._
