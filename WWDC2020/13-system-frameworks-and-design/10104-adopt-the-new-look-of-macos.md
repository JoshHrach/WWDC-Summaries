# Adopt the New Look of macOS
**WWDC20 · Session 10104** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10104/)

_Platforms:_ macOS Big Sur 11

## Overview
macOS Big Sur introduces the most dramatic visual redesign of macOS in years, featuring full-height sidebars with colorful icons, an overhauled toolbar system with multiple styles, new inset table view selections, large controls, system text styles, and the arrival of SF Symbols on the Mac. This session guides AppKit developers through the APIs needed to adopt this new look, noting what happens automatically vs. what requires explicit adoption.

Many design changes apply automatically when building against the Big Sur SDK: updated control appearance, the new toolbar borderless style, scroll-sensitive title bar separators, and symbol image mapping for existing named images. Key structural features — full-height sidebars, toolbar styles, split-view tracking toolbar items, and custom accent colors — require a small amount of adoption work but deliver significant visual improvement.

## Key Topics
- **Full-height sidebar** — Requires `NSSplitViewController` with sidebar `SplitViewItem` behavior plus the `fullSizeContentView` window mask; new `NSView` safe area APIs (`safeAreaInsets`, etc.) and Interface Builder Safe Area Layout Guide support the extended layout.
- **Sidebar tint customization** — New `NSOutlineViewDelegate.outlineView(_:tintConfigurationForItem:)` returns `NSTintConfiguration` (`.default`, `.monochrome`, `.preferredColor(color:)`, `.fixedColor(color:)`) for per-item icon tinting.
- **Toolbar styles** **[NEW]** — `NSWindow.toolbarStyle`: `.unified` (new default; large controls, inline title), `.unifiedCompact` (smaller height, optional inline title), `.preference` (auto-used by `NSTabViewController` with toolbar tab style), `.expanded` (legacy style with top title), `.automatic` (default; selects based on window structure).
- **`NSWindow.subtitle`** **[NEW]** — Secondary text below the window title in unified style, beside it in expanded style.
- **`NSToolbarItem.isNavigational`** **[NEW]** — Marks back/forward items; pins them to the leading edge of the title area; non-movable during customization.
- **`NSSearchToolbarItem`** **[NEW]** — Collapsible search toolbar item; set `searchField` property; automatically backwards-compatible (falls back to plain search field on older systems).
- **`NSTrackingSeparatorToolbarItem`** **[NEW]** — Toolbar item that tracks a specific `NSSplitView` divider index; creates visually distinct sections aligned with split view panes.
- **`NSSplitViewItem.allowsFullHeightLayout`** **[NEW]** — Opt out of full-height sidebar for specific items.
- **`NSSplitViewItem.titlebarSeparatorStyle`** **[NEW]** — Per-pane control of separator between toolbar and content (`.automatic`, `.line`, `.shadow`, `.none`).
- **Custom accent colors** — Define a named color in the asset catalog; specify it in the "Global Accent Color Name" build setting; applies when system is set to "Multicolor" accent preference.
- **Large control size** **[NEW]** — `NSControl.ControlSize.large` available for buttons, pop-ups, pull-downs, segmented controls, text fields, search fields; used automatically by unified toolbar.
- **`NSTableView.style`** **[NEW]** — `.automatic`, `.fullWidth`, `.inset`, `.sourceList`; `effectiveStyle` is read-only resolution of `.automatic`; replaces deprecated `selectionHighlightStyle` for source lists.
- **Text styles on macOS** **[NEW]** — `NSFont.preferredFont(forTextStyle:options:)` and `NSFontDescriptor.preferredFontDescriptor(forTextStyle:options:)` with `NSFont.TextStyle` values (.body, .headline, etc., calibrated for Mac's 13pt body).
- **SF Symbols on macOS** **[NEW]** — `NSImage(systemSymbolName:accessibilityDescription:)` — over 2,500 symbols available; `NSImageView.symbolFont` and `NSImageView.symbolImageScaling` for configuration; `NSImage.withSymbolConfiguration(_:)` for manual specialization; `NSImageSymbolConfiguration` init with size/weight/scale or text style.

## APIs & Frameworks

### AppKit
- **`NSWindow.toolbarStyle`** **[NEW]** — `NSWindow.ToolbarStyle` enum
- **`NSWindow.subtitle`** **[NEW]** — `String`; secondary window title
- **`NSToolbarItem.isNavigational`** **[NEW]** — `Bool`; pins to leading title area
- **`NSSearchToolbarItem`** **[NEW]** — `init(itemIdentifier:)`; `searchField: NSSearchField`
- **`NSTrackingSeparatorToolbarItem`** **[NEW]** — `init(itemIdentifier:splitView:dividerIndex:)`
- **`NSSplitViewItem.allowsFullHeightLayout`** **[NEW]** — `Bool`
- **`NSSplitViewItem.titlebarSeparatorStyle`** **[NEW]** — `NSTitlebarSeparatorStyle`
- **`NSTintConfiguration`** **[NEW]** — `.default`, `.monochrome`, `.preferredColor(color:)`, `.fixedColor(color:)`
- **`NSOutlineViewDelegate.outlineView(_:tintConfigurationForItem:)`** **[NEW]** — Returns optional `NSTintConfiguration` per item
- **`NSTableView.style`** **[NEW]** — `NSTableView.Style` enum (`.automatic`, `.fullWidth`, `.inset`, `.sourceList`)
- **`NSTableView.effectiveStyle`** **[NEW]** — Read-only resolved style
- **`NSControl.ControlSize.large`** **[NEW]** — Large control size constant
- **`NSFont.preferredFont(forTextStyle:options:)`** **[NEW]** — System font for a text style
- **`NSFontDescriptor.preferredFontDescriptor(forTextStyle:options:)`** **[NEW]**
- **`NSFont.TextStyle`** **[NEW]** — Enum: `.body`, `.headline`, `.subheadline`, `.footnote`, `.caption1`, `.caption2`, `.callout`, `.largeTitle`, `.title1`, `.title2`, `.title3`
- **`NSImage(systemSymbolName:accessibilityDescription:)`** **[NEW]** — System SF Symbol init
- **`NSImage.withSymbolConfiguration(_:)`** **[NEW]** — Returns specialized symbol image
- **`NSImageSymbolConfiguration`** **[NEW]** — `init(pointSize:weight:scale:)`, `init(textStyle:)`, `init(textStyle:scale:)`
- **`NSImageView.symbolFont`** **[NEW]** — `NSFont` for symbol rendering
- **`NSImageView.symbolImageScaling`** **[NEW]** — `NSImageScaling` for symbol images
- **`NSView` safe area APIs** **[NEW]** — `safeAreaInsets`, `safeAreaLayoutGuide`, `safeAreaRect`

## Code Highlights

Sidebar tint configuration:
```swift
func outlineView(_ outlineView: NSOutlineView, tintConfigurationForItem item: Any) -> NSTintConfiguration? {
    if let sectionItem = item as? SectionItem {
        return sectionItem.isSecondarySection ? .monochrome : .default
    }
    return nil  // inherit from parent
}
```

Collapsible search item adoption:
```swift
var searchItem = NSSearchToolbarItem(itemIdentifier: searchIdentifier)
searchItem.searchField = searchField  // reuse existing NSSearchField
```

Split view tracking toolbar separator:
```swift
let trackingItem = NSTrackingSeparatorToolbarItem(
    itemIdentifier: identifier, splitView: splitView, dividerIndex: 1)
```

System symbol image:
```swift
let newFolderImage = NSImage(systemSymbolName: "plus.rectangle.on.folder",
                             accessibilityDescription: "New Folder")
```

## Takeaways
- Most macOS Big Sur visual changes apply automatically when building against the new SDK; key structural features (full-height sidebar, toolbar style, split-tracking separators, custom accent color) require a small amount of explicit adoption.
- Use `NSWindow.toolbarStyle = .unified` for the new standard toolbar appearance with large controls and an inline title; `.preference` is used automatically by `NSTabViewController`-based preference windows.
- `NSTableView.style` replaces the deprecated `selectionHighlightStyle`; the `.inset` style is the new default for apps linked against the Big Sur SDK.
- SF Symbols have arrived on macOS with over 2,500 symbols; use `NSImageView` (not layer contents) for correct display and alignment; provide accessibility descriptions for all symbol images.

---
_Source: WWDC20 Session 10104 page (abstract, chapter summaries, code samples, and resource links)._
