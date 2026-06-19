# What's New in AppKit for macOS
**WWDC19 · Session 210** · [Watch](https://developer.apple.com/videos/play/wwdc2019/210/)

_Platforms:_ macOS Catalina 10.15

## Overview
AppKit in macOS Catalina 10.15 gains a wide range of improvements: new dynamic system colors and a programmatic color provider API, Extended Dynamic Range screen APIs for the new Pro Display XDR, NSColorSampler for on-screen pixel picking without a Screen Recording permission prompt, a fully new NSTextCheckingController to replace NSSpellChecker, enhanced toolbar and Touch Bar controls, NSSwitch, Compositional Layout and Diffable Data Sources for NSCollectionView, Apple Pencil / tablet event support via Sidecar, asynchronous NSWorkspace open methods, and main-thread-safe NSResponder deallocation.

The session also briefly introduces UIKit on Mac (Catalyst) and SwiftUI as two additional UI frameworks now available on macOS, directing developers to other sessions for deep dives. On the Foundation side, new geometry types using leading/trailing semantics, `NSRelativeDateFormatter`, `NSListFormatter`, and Combine are highlighted.

## Key Topics

**NSColor**
New teal and indigo system colors (dynamic, appearance-adaptive). `NSColor` now uses tagged pointers for zero-allocation color storage — `CGColor` derived from tagged-pointer colors must be used within the same autorelease pool. New `NSColorSampler.show(selectionHandler:)` picks any on-screen color without triggering Screen Recording consent. New `NSColor(name:dynamicProvider:)` initializer allows block-based programmatic color resolution per appearance.

**NSScreen: Extended Dynamic Range**
`NSScreen.localizedName` for human-readable display names. `maximumExtendedDynamicRangeColorComponentValue` (existing) and new `maximumPotentialExtendedDynamicRangeColorComponentValue` (reports headroom even before entering EDR mode). `maximumReferenceExtendedDynamicRangeColorComponentValue` reports the nit budget before reference-quality is compromised on the Pro Display XDR. `CAMetalLayer.preferredDevice` / `MTKView.preferredDevice` simplify GPU selection for the active display.

**Text and Fonts**
`NSTextView` adaptive color mapping for dark appearance. `NSTextCheckingController` replaces `NSSpellChecker` with support for grammar, data detection, and autocorrection; requires adopting `NSTextCheckingClient`. `NSFontDescriptor.withDesign(_:)` switches fonts to rounded, serif, or monospaced system variants. `NSAttributedString` encoding APIs now accept source/destination OS for automatic cross-platform font size adjustment. `NSLayoutManager.usesDefaultHyphenation` enables one-property hyphenation.

**Toolbar and Touch Bar**
`NSToolbarItem.isBordered` enables push-button style without a custom view. `NSToolbarItem.title` for text-labeled buttons. `NSToolbarItemGroup` gains convenience constructors and can now represent as segmented control, pull-down menu, or pop-up menu. `NSMenuToolbarItem` presents an `NSMenu` directly as a toolbar item. `NSTouchBar.isAutomaticCustomizeTouchBarMenuItemEnabled` class property. `NSStepper TouchBarItem` **[NEW]** — discrete stepper for dates, numbers, or tool selection. `NSSliderTouchBarItem.minimumSliderWidth` / `maximumSliderWidth`.

**Controls**
`NSSwitch` **[NEW]** — full NSControl subclass for heavy on/off toggles (supports bindings and formatters). `NSView.contentHuggingPriority` / `.contentCompressionResistancePriority` can be zeroed to skip intrinsic-size measurement in grid layouts. `NSResponder` now automatically moves `dealloc` to the main thread for itself and all subclasses, eliminating a class of background-thread dealloc crashes. `IBSegueAction` annotation for configuring view controllers during storyboard segue with custom initializer data.

**NSCollectionView**
Compositional Layout **[NEW]** — no subclassing required; supports container-relative sizing, layout breaks, nestable groups, per-section scrolling. Diffable Data Sources **[NEW]** — identifier-based; automatic animation inference; eliminates `performBatchUpdates` and `reloadData`.

**Apple Pencil / Tablet Events via Sidecar**
Sidecar turns iPads into tablet displays. `NSEvent.subtype == .tabletPoint` indicates pressure-capable input. `NSEvent.pressure` provides per-event pressure. New `NSEvent.EventType.changeMode` and corresponding `NSResponder.changeMode(_:)` for double-tap Apple Pencil tool switching. Local event monitor pattern for tool cycling.

**NSWorkspace**
New async open methods: `open(_:options:completionHandler:)` and `openApplication(at:options:configuration:completionHandler:)`. `NSWorkspace.OpenConfiguration` **[NEW]** — controls user prompts, Recents entry, hide-on-launch, foreground/background activation.

**Open/Save Panels**
Now run in a separate process (previously sandboxed apps only). Generally transparent; subclasses relying on specific view hierarchies may need adjustment.

**Foundation**
`NSDirectionalRectEdge`, `NSDirectionalEdgeInsets`, `NSRectAlignment` — leading/trailing geometry types that automatically flip for RTL. `NSRelativeDateFormatter` **[NEW]** — formats relative time ("last week" / "1 week ago"). `NSListFormatter` **[NEW]** — formats arrays of objects with correct comma placement and Oxford comma/conjunction handling.

## APIs & Frameworks

**AppKit**
- `NSColor.systemTeal`, `NSColor.systemIndigo` **[NEW]**
- `NSColor(name:dynamicProvider:)` **[NEW]**
- `NSColorSampler.show(selectionHandler:)` **[NEW]**
- `NSScreen.localizedName` **[NEW]**
- `NSScreen.maximumPotentialExtendedDynamicRangeColorComponentValue` **[NEW]**
- `NSScreen.maximumReferenceExtendedDynamicRangeColorComponentValue` **[NEW]**
- `CAMetalLayer.preferredDevice` **[NEW]**; `MTKView.preferredDevice` **[NEW]**
- `NSTextCheckingController` **[NEW]**; `NSTextCheckingClient` protocol **[NEW]**
- `NSFontDescriptor.withDesign(_:)` **[NEW]** — `.rounded`, `.serif`, `.monospaced`
- `NSAttributedString` cross-platform font size encoding APIs **[NEW]**
- `NSLayoutManager.usesDefaultHyphenation` **[NEW]**
- `NSToolbarItem.isBordered` **[NEW]**; `NSToolbarItem.title` **[NEW]**
- `NSToolbarItemGroup` convenience constructors, segmented/menu representations **[NEW]**
- `NSMenuToolbarItem` **[NEW]**
- `NSTouchBar.isAutomaticCustomizeTouchBarMenuItemEnabled` (class property) **[NEW]**
- `NSStepperTouchBarItem` **[NEW]**
- `NSSliderTouchBarItem.minimumSliderWidth` **[NEW]**; `maximumSliderWidth` **[NEW]**
- `NSSwitch` **[NEW]** — full NSControl subclass
- `NSView.horizontalContentSizeConstraintActive` / `verticalContentSizeConstraintActive` **[NEW]**
- `NSCollectionViewCompositionalLayout` **[NEW]**
- `NSCollectionViewDiffableDataSource` **[NEW]**; `NSDiffableDataSourceSnapshot` **[NEW]**
- `NSEvent.EventType.changeMode` **[NEW]**; `NSResponder.changeMode(_:)` **[NEW]**
- `NSWorkspace.OpenConfiguration` **[NEW]**
- `NSWorkspace.open(_:options:completionHandler:)` **[NEW]**
- `NSWorkspace.openApplication(at:options:configuration:completionHandler:)` **[NEW]**

**Foundation**
- `NSDirectionalRectEdge`, `NSDirectionalEdgeInsets`, `NSRectAlignment` **[NEW]**
- `NSRelativeDateFormatter` **[NEW]**
- `NSListFormatter` **[NEW]**
- Combine framework **[NEW]** (see Sessions 721, 722)

## Code Highlights

Dynamic color provider:

```swift
let brandColor = NSColor(name: "BrandPrimary") { appearance in
    switch appearance.bestMatch(from: [.aqua, .darkAqua]) {
    case .darkAqua: return NSColor(red: 0.2, green: 0.4, blue: 0.9, alpha: 1)
    default:        return NSColor(red: 0.1, green: 0.3, blue: 0.8, alpha: 1)
    }
}
```

Tablet event handling for Apple Pencil via Sidecar:

```swift
override func mouseDragged(with event: NSEvent) {
    if event.subtype == .tabletPoint {
        let pressure = event.pressure   // 0.0 – 1.0
        strokeWidth = baseWidth * pressure
    }
    // draw ...
}
```

IBSegueAction:

```swift
@IBSegueAction
func showPetDetails(_ coder: NSCoder) -> PetDetailViewController? {
    PetDetailViewController(coder: coder, petName: selectedPetName)
}
```

## Takeaways
- `NSResponder` subclasses (including all views) now safely dealloc on the main thread — a common crash category is eliminated without any code changes.
- Compositional Layout and Diffable Data Sources bring the same modern collection view infrastructure from iOS 13 to `NSCollectionView`, dramatically reducing boilerplate.
- `NSColorSampler` replaces custom screen-pixel-picking code without triggering the Screen Recording privacy prompt.
- With Sidecar shipping in macOS Catalina, many more Mac users will have Apple Pencil access — apps with drawing surfaces should handle tablet pressure events now.

---
_Source: WWDC19 Session 210 page (abstract, transcript, and resource links)._
