# Modernize Your AppKit App
**WWDC26 · Session 289** · [Watch](https://developer.apple.com/videos/play/wwdc2026/289/)

_Platforms:_ macOS

## Overview
This session covers three areas where AppKit apps can be brought in line with modern macOS conventions: input handling, continuity across launches, and visual design. The central theme is harmony — the app's input model, lifecycle, and appearance should all feel integrated with the rest of the system rather than operating in isolation.

The session works through a structured progression: replacing `mouseDown:` overrides and tracking loops with gesture recognizers and control events, enabling keyboard navigation, implementing graceful quit behavior and full state restoration with `NSWindowRestoration`, and adopting the new Liquid Glass visual updates and corner concentricity API introduced in macOS 27.

## Key Topics

### Modern Input (1:06 – 5:50)
`mouseDown:` overrides and event-tracking loops are the legacy way to handle mouse input. Modern replacements:
- **Selection** — observe the `.selected` property on `NSCollectionView` / `NSTableView` items instead of intercepting clicks
- **Context menus** — implement `menuForEvent:` or assign `.menu` on the view
- **Drag and drop** — use `NSTableViewDataSource.tableView(_:pasteboardWriterForRow:)` and related pasteboard delegate methods instead of `mouseDragged:`
- **Text selection** — **[NEW]** `NSTextSelectionManager` brings full macOS text selection (bidirectional, drag, toggle) to any custom view outside `NSTextView`
- **Control events** — `NSControl.addTarget(_:action:for:)` with typed `NSControl.Events` values (e.g., `.trackingEndedOutside`) replaces raw event monitoring
- **Custom interactions** — `NSGestureRecognizer` subclasses handle mouse, trackpad, Force Click, and Sidecar touch uniformly; `hitTest(_:) -> NSView?` returns `nil` to opt out of hit-testing when needed

### Keyboard Navigation (5:51 – 8:56)
- `window.autorecalculatesKeyViewLoop = true` — the window recalculates Tab order automatically as the view hierarchy changes; eliminates manual `recalculateKeyViewLoop()` calls
- Status items with custom popover UI should use **[NEW]** `NSStatusItem.expandedInterfaceDelegate` and `NSStatusItemExpandedInterfaceSession` — AppKit manages keyboard focus, dismiss-on-click-outside, and the system menu bar key state; call `session.cancel()` to request dismissal from your own code

### Continuity Across Launches (8:57 – 14:08)
A great Mac app quits instantly and relaunches exactly where it was.

**Graceful termination**: `window.preventsApplicationTerminationWhenModal = false` on non-critical sheets lets the app terminate immediately during system shutdown/restart.

**State restoration** (`NSWindowRestoration`):
1. Set `window.identifier`, `window.setFrameAutosaveName`, `window.isRestorable = true`, and `window.restorationClass`
2. Override `encodeRestorableState(with:)` to save state (e.g., selected item UUID) using `NSCoder.encode(_:forKey:)`
3. Call `invalidateRestorableState()` whenever the state changes
4. In the `NSWindowRestoration` class, implement `restoreWindow(withIdentifier:state:completionHandler:)` to recreate and return windows by identifier
5. Override `restoreState(with:)` to read the saved values and restore the UI

### Design Updates (14:09 – 16:59)
- **Liquid Glass** updates apply automatically in macOS 27 to sidebars, scroll edge effects, toolbar items, and controls; a new interactive glass effect gives controls physical tactile feedback on click
- **[NEW]** `NSViewCornerConfiguration` — override `cornerConfiguration` on an `NSView` subclass to have the view's corners adapt to its container's radius using `.containerConcentric(minimumCornerRadius)` via `NSViewCornerRadius.containerConcentric(_:)` and `NSViewCornerConfiguration.uniformCorners(radius:)`

## APIs & Frameworks

**AppKit — Input**
- `NSGestureRecognizer` — preferred over `mouseDown:` for all custom interactions
- `NSGestureRecognizer.state` — `.ended`, `.possible`
- `NSControl.addTarget(_:action:for:)` with `NSControl.Events`
- `NSControl.Events.trackingEndedOutside` — example typed event
- **[NEW]** `NSTextSelectionManager` — text selection in custom views
- `NSTableViewDataSource.tableView(_:pasteboardWriterForRow:)` — modern drag source
- `NSPasteboardItem` — modern pasteboard writer
- `NSView.hitTest(_:) -> NSView?` override — opt out of hit testing
- `NSView.menuForEvent(_:)` — context menu from event

**AppKit — Status Items**
- **[NEW]** `NSStatusItem.expandedInterfaceDelegate`
- **[NEW]** `NSStatusItemExpandedInterfaceDelegate` protocol
  - `statusItem(_:didBegin:)` — show custom window
  - `statusItemDidEndExpandedInterfaceSession(_:animated:)` — hide window
- **[NEW]** `NSStatusItemExpandedInterfaceSession`
- `NSStatusItemExpandedInterfaceSession.cancel()` — request dismissal

**AppKit — Keyboard Navigation**
- `NSWindow.autorecalculatesKeyViewLoop` — **[NEW flag / updated behavior]** auto-recalculate Tab order

**AppKit — State Restoration**
- `NSWindowRestoration` protocol — `restoreWindow(withIdentifier:state:completionHandler:)`
- `NSWindow.identifier` — `NSUserInterfaceItemIdentifier`
- `NSWindow.setFrameAutosaveName(_:)`
- `NSWindow.isRestorable`
- `NSWindow.restorationClass`
- `NSResponder.encodeRestorableState(with:)` override
- `NSResponder.restoreState(with:)` override
- `NSResponder.invalidateRestorableState()`
- `NSCoder.encode(_:forKey:)` / `NSCoder.decodeObject(of:forKey:)`
- `NSWindow.preventsApplicationTerminationWhenModal` — **[NEW]** set `false` for non-critical sheets

**AppKit — Design**
- **[NEW]** `NSView.cornerConfiguration` — computed property override
- **[NEW]** `NSViewCornerConfiguration` — `uniformCorners(radius:)`
- **[NEW]** `NSViewCornerRadius.containerConcentric(_:)` — matches container corner radius

## Code Highlights

Auto-recalculate key view loop:
```swift
window.autorecalculatesKeyViewLoop = true
```

Allow graceful termination during modal:
```swift
window.preventsApplicationTerminationWhenModal = false
```

Save and restore selection state:
```swift
override func encodeRestorableState(with coder: NSCoder) {
    super.encodeRestorableState(with: coder)
    coder.encode(selectedProduct?.identifier.uuid,
                 forKey: RestorationKeys.productIdentifier)
}
```

Concentric corner radius in a custom view:
```swift
override var cornerConfiguration: NSViewCornerConfiguration? {
    let radius: NSViewCornerRadius = .containerConcentric(minimumCornerRadius)
    return .uniformCorners(radius: radius)
}
```

## Takeaways
- Replace all `mouseDown:` overrides and tracking loops with `NSGestureRecognizer` or typed control events; this also gains Sidecar touch support for free.
- Enable `autorecalculatesKeyViewLoop` on every window — if your app's view hierarchy changes at runtime, Tab navigation will otherwise break silently.
- Implement full `NSWindowRestoration` (encode, invalidate, restore) — even a basic implementation covering which documents and selections were open dramatically improves relaunch quality.
- Adopt `NSViewCornerConfiguration.uniformCorners(radius: .containerConcentric(_:))` wherever views are visually nested inside a container with a rounded corner to maintain the Liquid Glass aesthetic.

---
_Source: WWDC26 Session 289 page (abstract, chapter summaries, code samples, and resource links)._
