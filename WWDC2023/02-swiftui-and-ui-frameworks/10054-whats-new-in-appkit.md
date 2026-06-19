# What's New in AppKit
**WWDC23 · Session 10054** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10054/)

_Platforms:_ macOS Sonoma 14

## Overview
This session surveys the significant new APIs and behavioral changes in AppKit for macOS Sonoma. It covers improvements across seven areas: controls (table column menus, progress indicator integration, button bezel styles, inspectors, popovers), menus (completely rewritten in Cocoa with palette menus, section headers, badges), cooperative app activation, graphics (NSBezierPath/CGPath bridging, CADisplayLink, new system fill colors, view clipping changes), images and symbols (symbol effects, HDR support, asset catalog code generation), text input (new insertion indicator, language-sensitive line breaking), and Swift/SwiftUI integration improvements.

A particularly impactful change is that most `NSView`s no longer clip to their bounds by default when linked against the macOS Sonoma SDK, removing a long-standing source of clipped glyphs and shadows. Another major addition is `CADisplayLink` on macOS, enabling display-synchronized animation for AppKit apps.

## Key Topics

### Controls
- **NSTableView column customization**: New `tableView(_:userCanChangeVisibilityOf:)` delegate method; AppKit handles menu creation, localization, and state restoration on relaunch.
- **NSProgressIndicator.observedProgress**: New property accepting a `Foundation.Progress` object; automatically updates the indicator from any thread.
- **NSButton bezel style `automatic`** **[NEW]**: Default for all button initializers; adapts to push, toolbar, or flexible push style based on context. Existing bezel style names renamed to semantic identifiers (e.g., "Recessed" → "Accessory Bar"). Discouraged styles deprecated.
- **NSSplitViewItem inspector** **[NEW]**: Full-height trailing split view item; `NSSplitViewItem(inspectorWithViewController:)` initializer; back-deploys to macOS Big Sur. Add `NSToolbarItem.Identifier.toggleInspector` and `inspectorTrackingSeparator` to toolbar.
- **NSPopover**: New `show(relativeTo: NSToolbarItem)` method for toolbar-anchored popovers; new `hasFullSizeContent` property to extend content into the chevron area.

### Menus (New Cocoa Implementation)
- Menus completely rewritten in Cocoa, reducing memory and CPU usage.
- **Section headers**: `NSMenuItem.sectionHeader(title:)` class method **[NEW]**.
- **Palette menus** **[NEW]**: Set `menu.presentationStyle = .palette`; lay out items horizontally. Set `offStateImage`/`onStateImage` or rely on automatic tint for template images.
- **Selection modes for palette menus** **[NEW]**: `.selectAny` (toggle individual) or `.selectOne` (radio). `menu.selectedItems` property to get/set selected items. Groups are formed by items sharing the same target/action pair — also works in regular menus.
- **Convenience palette constructor**: `NSMenu(title:palette:colors:titles:template:selectionHandler:)` **[NEW]**.
- **Menu item badges** **[NEW]**: String, count, or specialized types (`newItems`, `alerts`, `updates`) with automatic text and localization.

### Cooperative App Activation
- `activate(ignoringOtherApps:)` and `activateIgnoringOtherApps` option **[DEPRECATED]**.
- New `NSApplication.activate()` and `NSRunningApplication.activate(options:)` — activation is now a request the system evaluates in context.
- **`NSApplication.yieldActivation(to:)`** / **`NSRunningApplication.yieldActivation()`** **[NEW]**: Active app yields to a target app before the target requests activation. NSWorkspace handles this automatically for `openURL`/`openApplication`.

### Graphics
- **NSBezierPath ↔ CGPath bridging** **[NEW]**: `NSBezierPath(cgPath:)` init and `.cgPath` property (copies, not toll-free bridged).
- **CADisplayLink on macOS** **[NEW]**: Obtain from `NSView.displayLink(target:selector:)`, `NSWindow.displayLink(target:selector:)`, or `NSScreen.displayLink(target:selector:)`. Automatically tracks display changes and suspends when not on a display.
- **New system fill colors** **[NEW]**: `NSColor.systemFill`, `.secondarySystemFill`, `.tertiarySystemFill`, `.quaternarySystemFill`, `.quinarySystemFill` — dynamic, adapt to Dark Mode and Increased Contrast.
- **NSView no longer clips to bounds by default** **[NEW behavior]**: Most views do not clip when linked on macOS Sonoma. `NSView.clipsToBounds` property available back to macOS Mavericks. `dirtyRect` may now extend beyond bounds; use it to determine what to draw, not where.

### Images and Symbols
- **Symbol effects** **[NEW]**: `NSImageView.addSymbolEffect(_:options:)` — bounce, replacement, pulse, variableColor, etc.
- **Asset catalog locale support**: Images and symbols in asset catalogs now auto-adapt to system locale (like SF Symbols since Ventura). `NSImage.image(locale:)` method for fixed locale.
- **NSImageView HDR support** **[NEW]**: Images with HDR content displayed in HDR on EDR-capable hardware. `NSImageView.preferredImageDynamicRange` to override.
- **Asset catalog code generation** (Xcode 15): Images and colors as static `NSImage`/`NSColor` properties; non-optional, compiler-checked.

### Text Improvements
- **NSTextInsertionIndicator** **[NEW]**: New view for custom text views; provides the system insertion indicator with accent color and dictation glow. Set `displayMode = .hidden` on resign first responder.
- **Cursor accessory**: Displays input mode, dictation state, caps lock state below insertion point; pins to bottom of document if off-screen.
- **Language-sensitive line breaking**: Title/Headline text styles in Korean no longer break within words; German title text fields auto-hyphenate at morpheme boundaries when hyphenation is disabled.

### Swift and SwiftUI
- **`Sendable` conformance** **[NEW]**: `NSColor`, `NSShadow`, and other thread-safe AppKit classes now conform to `Sendable`.
- **`Transferable` conformance** **[NEW]**: `NSImage`, `NSColor`, `NSSound` conform to `Transferable`, enabling Drag & Drop and Sharing in SwiftUI views.
- **`@ViewLoading` / `@WindowLoading`** (macOS Ventura 13.3+): Removes optionality from properties initialized in `loadView`/`loadWindow`.
- **`#Preview` macro for AppKit** (Xcode 15) **[NEW]**: Preview `NSView` and `NSViewController` using the same `#Preview` macro as SwiftUI.
- **NSHostingView/NSHostingController scene bridging** **[NEW]**: SwiftUI modifiers like `toolbar` and `navigationTitle` now bridge to the enclosing `NSWindow`; `sceneBridgingOptions` property for explicit control.

## APIs & Frameworks

- `NSTableView` — `tableView(_:userCanChangeVisibilityOf:)` delegate method **[NEW]**
- `NSProgressIndicator.observedProgress` **[NEW]** — `Progress` binding
- `NSButton.BezelStyle.automatic` **[NEW]** — adaptive default bezel style
- `NSSplitViewItem(inspectorWithViewController:)` **[NEW]**
- `NSToolbarItem.Identifier.toggleInspector` **[NEW]**
- `NSToolbarItem.Identifier.inspectorTrackingSeparator` **[NEW]**
- `NSPopover.show(relativeTo: NSToolbarItem)` **[NEW]**
- `NSPopover.hasFullSizeContent` **[NEW]**
- `NSMenuItem.sectionHeader(title:)` **[NEW]**
- `NSMenu.presentationStyle` **[NEW]** — `.palette`, `.regular`
- `NSMenu.selectionMode` **[NEW]** — `.selectAny`, `.selectOne`
- `NSMenu.selectedItems` **[NEW]**
- `NSMenu(title:palette:colors:titles:template:selectionHandler:)` **[NEW]**
- `NSMenuItemBadge` **[NEW]** — `.string(_:)`, `.count(_:)`, `.newItems`, `.alerts`, `.updates`
- `NSApplication.activate()` **[NEW]**
- `NSRunningApplication.activate(options:)` **[NEW]**
- `NSApplication.yieldActivation(to:)` **[NEW]**
- `NSApplication.activate(ignoringOtherApps:)` **[DEPRECATED]**
- `NSBezierPath.init(cgPath:)` **[NEW]**
- `NSBezierPath.cgPath` **[NEW]**
- `CADisplayLink` — now available on macOS **[NEW]**
- `NSView.displayLink(target:selector:)` **[NEW]**
- `NSWindow.displayLink(target:selector:)` **[NEW]**
- `NSScreen.displayLink(target:selector:)` **[NEW]**
- `NSColor.systemFill` **[NEW]** and `.secondarySystemFill`, `.tertiarySystemFill`, `.quaternarySystemFill`, `.quinarySystemFill`
- `NSView.clipsToBounds` — now `false` by default for most views on macOS Sonoma **[behavior change]**
- `NSImageView.addSymbolEffect(_:options:)` **[NEW]**
- `NSImageView.preferredImageDynamicRange` **[NEW]**
- `NSTextInsertionIndicator` **[NEW]**
- `NSTextInsertionIndicator.displayMode` **[NEW]**
- `Sendable` conformance — `NSColor`, `NSShadow`, and other AppKit types **[NEW]**
- `Transferable` conformance — `NSImage`, `NSColor`, `NSSound` **[NEW]**
- `@ViewLoading` property wrapper — `NSViewController` (macOS Ventura 13.3+)
- `@WindowLoading` property wrapper — `NSWindowController`
- `#Preview` macro for `NSView`/`NSViewController` (Xcode 15) **[NEW]**
- `NSHostingView.sceneBridgingOptions` **[NEW]**
- `NSHostingController.sceneBridgingOptions` **[NEW]**

## Code Highlights

```swift
// Table column customization (3 lines)
func tableView(_ tableView: NSTableView, userCanChangeVisibilityOf column: NSTableColumn) -> Bool {
    return column.identifier != "Name"
}

// Progress indicator binding
progressIndicator.observedProgress = task.progress

// Inspector split view
let inspectorItem = NSSplitViewItem(inspectorWithViewController: inspectorVC)
splitViewController.addSplitViewItem(inspectorItem)

// CADisplayLink from a view
let displayLink = view.displayLink(target: self, selector: #selector(stepAnimation))
displayLink.add(to: .main, forMode: .common)

// Symbol effect
wifiImageView.image = NSImage(systemSymbolName: "wifi", accessibilityDescription: "wifi")
wifiImageView.addSymbolEffect(.variableColor.iterative, options: .repeating)

// Palette menu
let menu = NSMenu(title: "Colors", palette: colors, titles: titles) { menu in
    print(menu.selectedItems)
}
menu.selectionMode = .selectOne
```

## Takeaways
- `NSView.clipsToBounds` is now `false` by default on macOS Sonoma — audit drawing code that relies on implicit clipping, especially any that fills `dirtyRect` with a background color.
- `CADisplayLink` is finally available on macOS, providing display-synchronized animation without the complexity of CVDisplayLink.
- The new Cocoa menu implementation unlocks palette menus and section headers as first-class, single-line-of-code features.
- `NSImage`, `NSColor`, and `NSSound` now conform to `Transferable`, making it straightforward to use SwiftUI Drag & Drop and Sharing APIs in AppKit apps.

---
_Source: WWDC23 Session 10054 page (abstract, chapter summaries, code samples, and resource links)._
