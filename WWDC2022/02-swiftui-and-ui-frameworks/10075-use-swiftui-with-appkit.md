# Use SwiftUI with AppKit
**WWDC22 · Session 10075** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10075/)

_Platforms:_ macOS Ventura 13

## Overview
This session, presented by an engineer from the Shortcuts team, explains how to incrementally adopt SwiftUI inside an existing AppKit Mac app. It covers five key integration areas: hosting SwiftUI views in AppKit view controllers, managing shared state between the two frameworks, placing SwiftUI inside collection and table view cells efficiently, controlling layout and sizing of hosted views, participating in the responder chain and focus system, and hosting legacy AppKit views inside a SwiftUI layout using the `NSViewRepresentable` protocol.

All examples are drawn directly from the macOS Shortcuts app, which arrived in macOS Monterey and blends SwiftUI and AppKit extensively to share views with iOS and watchOS while customising the Mac experience.

## Key Topics

### Hosting SwiftUI in AppKit
`NSHostingController` wraps a SwiftUI view as a standard `NSViewController`, making it composable in any existing view controller hierarchy. In Shortcuts, the sidebar `List` view is wrapped in an `NSHostingController` and added to an `NSSplitViewController` as a split-view item.

### Sharing State Between AppKit and SwiftUI
State that needs to be observed by both frameworks should live in a class conforming to `ObservableObject`. AppKit code subscribes to published properties using Combine; SwiftUI reads them with `@ObservedObject` or `@EnvironmentObject`. The session introduces a `SelectionModel` class with a `@Published selectedItem` that the split view controller observes via `sink` while the SwiftUI sidebar reads it as a binding.

### SwiftUI in Collection and Table View Cells
Cell reuse requires avoiding repeated add/remove of subviews. The recommended pattern: keep one `NSHostingView` per cell; on recycle, update its `rootView` property rather than creating a new hosting view. SwiftUI then diffs and reuses the existing view tree (e.g., the `VStack`/`Spacer` structure) and only updates changed data.

### Layout and Sizing
Hosting views and controllers expose intrinsic sizes as Auto Layout constraints automatically generated from the SwiftUI view's ideal, minimum, and maximum sizes. When placed as a window's `contentView`, they drive the window's min/max size automatically. For modal presentation, use `present(_:asPopoverRelativeTo:...)`, `presentAsSheet(_:)`, or `presentAsModalWindow(_:)`. New in macOS Ventura: `NSHostingController.sizingOptions` and `NSHostingView.sizingOptions` let you select which constraints are generated (`.minSize`, `.intrinsicContentSize`, `.maxSize`, `.preferredContentSize`).

### Responder Chain and Focus
SwiftUI views hosted in AppKit participate in the responder chain. Use `.focusable()` to make a view keyboard-focusable. Common command handlers: `.copyable`, `.cuttable`, `.pasteDestination` for clipboard operations; `.onMoveCommand`, `.onExitCommand` for arrow keys and Escape; `.onCommand(#selector(...))` for any AppKit selector or custom action. Always test with Full Keyboard Navigation both on and off.

### Hosting AppKit Views in SwiftUI (NSViewRepresentable)
Implement `NSViewRepresentable` (or `NSViewControllerRepresentable`) to embed an `NSView` into a SwiftUI layout. Lifecycle: `makeCoordinator()` → `makeNSView(context:)` → repeated `updateNSView(_:context:)` calls → `dismantleNSView(_:coordinator:)`. The coordinator is a persistent object used for delegation, KVO, and notifications. In `updateNSView`, compare old and new values before applying changes to avoid unnecessary work.

## APIs & Frameworks

**SwiftUI / AppKit Integration**
- `NSHostingController(rootView:)` — wraps a SwiftUI view as `NSViewController`
- `NSHostingView(rootView:)` — wraps a SwiftUI view as `NSView`
- `NSHostingView.rootView` — update to change displayed content in-place (cell reuse pattern)
- `NSHostingController.sizingOptions` **[NEW]** — controls which Auto Layout constraints are generated
- `NSHostingView.sizingOptions` **[NEW]** — same, for bare hosting views
- `NSViewRepresentable` protocol — hosts an `NSView` in SwiftUI
  - `makeCoordinator() -> Coordinator`
  - `makeNSView(context:) -> NSView`
  - `updateNSView(_:context:)`
  - `dismantleNSView(_:coordinator:)` (optional)
- `NSViewControllerRepresentable` protocol — hosts an `NSViewController` in SwiftUI

**SwiftUI Focus and Commands**
- `.focusable()` — makes a view keyboard-focusable
- `.copyable { ... }` — copy command handler
- `.cuttable { ... }` — cut command handler
- `.pasteDestination(payloadType:action:)` — paste command handler
- `.onMoveCommand { direction in ... }` — arrow-key handler
- `.onExitCommand { ... }` — Escape key handler
- `.onCommand(_:action:)` — handles any AppKit selector or custom selector

**Combine / State Sharing**
- `ObservableObject` + `@Published` — observable model object
- `Publisher.sink(receiveValue:)` — AppKit subscribes to SwiftUI model changes

## Code Highlights

Hosting a SwiftUI sidebar in an `NSSplitViewController`:
```swift
let sidebar = NSHostingController(rootView: SidebarView(model: selectionModel))
let splitViewItem = NSSplitViewItem(viewController: sidebar)
splitViewController.addSplitViewItem(splitViewItem)

// AppKit observes selection changes
cancellable = selectionModel.$selectedItem.sink { newItem in
    // update the NSSplitViewController detail column
}
```

Efficient SwiftUI hosting in a collection view cell (reuse pattern):
```swift
class ShortcutItemView: NSCollectionViewItem {
    private var hostingView: NSHostingView<ShortcutView>?

    func displayShortcut(_ shortcut: Shortcut) {
        let shortcutView = ShortcutView(shortcut: shortcut)
        if let hostingView {
            hostingView.rootView = shortcutView   // reuse, no subview churn
        } else {
            let newHostingView = NSHostingView(rootView: shortcutView)
            view.addSubview(newHostingView)
            setupConstraints(for: newHostingView)
            self.hostingView = newHostingView
        }
    }
}
```

`NSViewRepresentable` wrapping a custom `ScriptEditorView`:
```swift
struct ScriptEditorRepresentable: NSViewRepresentable {
    @Binding var sourceCode: String

    func makeNSView(context: Context) -> ScriptEditorView {
        let editor = ScriptEditorView(frame: .zero)
        editor.delegate = context.coordinator
        return editor
    }

    func updateNSView(_ nsView: ScriptEditorView, context: Context) {
        if sourceCode != nsView.sourceCode { nsView.sourceCode = sourceCode }
        nsView.isEditable = context.environment.isEnabled
        context.coordinator.representable = self
    }

    func makeCoordinator() -> Coordinator { Coordinator(representable: self) }

    class Coordinator: NSObject, ScriptEditorViewDelegate {
        var representable: ScriptEditorRepresentable
        func sourceCodeDidChange(in view: ScriptEditorView) {
            representable.sourceCode = view.sourceCode
        }
    }
}
```

Controlling sizing options (new in macOS Ventura):
```swift
hostingController.sizingOptions = [.minSize, .intrinsicContentSize, .maxSize]
```

## Takeaways
- Use `NSHostingController` / `NSHostingView` to embed SwiftUI anywhere in an existing AppKit hierarchy; shared state belongs in an `ObservableObject` that both sides can observe.
- In collection and table view cells, keep one `NSHostingView` per cell and update `rootView` on recycle — never add/remove subviews during scroll.
- New in macOS Ventura, `sizingOptions` on hosting controllers and views lets you tune which Auto Layout constraints are auto-generated, improving performance and preventing constraint conflicts.
- Implement `NSViewRepresentable` to embed any `NSView` in SwiftUI; use the coordinator for delegation and always guard `updateNSView` with equality checks to avoid unnecessary reloads.

---
_Source: WWDC22 Session 10075 page (abstract, transcript, and code samples)._
