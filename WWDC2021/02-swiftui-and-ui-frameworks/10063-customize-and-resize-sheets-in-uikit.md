# Customize and Resize Sheets in UIKit
**WWDC21 · Session 10063** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10063/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
iOS 15 adds significant customization to `UISheetPresentationController`, the presentation controller behind sheet-style modal presentations introduced in iOS 13. New capabilities include a half-height medium detent, configurable dimming removal to enable non-modal interactions, programmatic animated detent selection, scroll-expand control, grabber visibility, corner radius customization, a compact-height landscape appearance, and a new `adaptiveSheetPresentationController` property for building popovers that adapt to customized sheets in compact size classes.

The session walks through evolving a standard `PHPickerViewController` sheet into a persistent, non-modal, half-height photo library that remains visible while the user sees their work, demonstrating every key API step-by-step.

## Key Topics

### UISheetPresentationController
Access via `viewController.sheetPresentationController` (returns non-nil when `modalPresentationStyle` is `.pageSheet` or `.formSheet`, which is the default). Set properties before calling `present(_:animated:)`. This mirrors the pattern of `popoverPresentationController`.

### Detents
A **detent** is a height at which a sheet naturally rests. iOS 15 introduces two system detents:
- `.medium()` — approximately half the fully expanded sheet frame
- `.large()` — the full height of the sheet (prior to iOS 15, the only option)

Set `sheet.detents` to an array containing one or both. Combinations:
- `[.large()]` — default; standard full-height modal sheet
- `[.medium(), .large()]` — resizable between half and full height
- `[.medium()]` — fixed half-height sheet, no resize to full

### Scroll-Expand Behavior
By default (`prefersScrollingExpandsWhenScrolledToEdge = true`), scrolling a scroll view inside a medium-height sheet to its top expands the sheet to full height. Set this to `false` to require explicit drag on the grabber bar to expand — useful when the background content should remain continuously visible.

### Programmatic Animated Detent Changes
Set `sheet.selectedDetentIdentifier` to `.medium` or `.large` to jump to a detent. Wrap in `sheet.animateChanges { }` to animate the transition with the standard system animation curve. This also animates stacked sheets (e.g., the root sheet scales back up while the presented sheet collapses).

### Removing Dimming (Non-Modal UI)
`smallestUndimmedDetentIdentifier` controls which detents show the dimming view:
- `nil` (default) — dim at all detents
- `.medium` — no dim at medium; dim fades in when expanded to large

Removing the dim also enables touch-through: the user can interact with content behind the sheet at non-dimmed detents, enabling genuinely non-modal experiences where content in and behind the sheet are simultaneously usable.

### Visual Customizations
- `prefersGrabberVisible = true` — shows a visual grabber handle above the sheet content, useful when scroll-expand is disabled to indicate resizability
- `preferredCornerRadius = 20.0` — custom corner radius; system automatically adjusts stacked sheets' corners to stay consistent
- `prefersEdgeAttachedInCompactHeight = true` — iPhone landscape alternate: sheet attaches only at bottom edge rather than going full screen
- `widthFollowsPreferredContentSizeWhenEdgeAttached = true` — makes sheet width follow `presentedViewController.preferredContentSize` in landscape edge-attached mode

### Popover-to-Sheet Adaptation
For iPad, set `modalPresentationStyle = .popover` and configure `popoverPresentationController`. Use the new `popover.adaptiveSheetPresentationController` property to access the `UISheetPresentationController` the popover adapts to in compact size classes. Configure it with the same detents and properties as a direct sheet. In delegate callbacks, always read `adaptiveSheetPresentationController` via `popoverPresentationController?.adaptiveSheetPresentationController` to handle both regular and compact environments.

## APIs & Frameworks

**UIKit** (`import UIKit`) — **[NEW iOS 15]**

- `UISheetPresentationController : UIPresentationController` **[NEW iOS 15]** — sheet customization controller
  - `detents: [UISheetPresentationController.Detent]` **[NEW]** — array of allowed rest heights; default `[.large()]`
  - `UISheetPresentationController.Detent.medium()` **[NEW]** — half-height detent factory
  - `UISheetPresentationController.Detent.large()` **[NEW]** — full-height detent factory
  - `selectedDetentIdentifier: UISheetPresentationController.Detent.Identifier?` **[NEW]** — current detent; set to programmatically jump
  - `animateChanges(_ changes: () -> Void)` **[NEW]** — animate detent/property changes
  - `prefersScrollingExpandsWhenScrolledToEdge: Bool` **[NEW]** — default `true`; set `false` to prevent scroll-driven expansion
  - `smallestUndimmedDetentIdentifier: UISheetPresentationController.Detent.Identifier?` **[NEW]** — smallest undimmed detent; `nil` dims all
  - `prefersGrabberVisible: Bool` **[NEW]** — show/hide grabber handle
  - `preferredCornerRadius: CGFloat?` **[NEW]** — custom corner radius
  - `prefersEdgeAttachedInCompactHeight: Bool` **[NEW]** — alternate landscape appearance
  - `widthFollowsPreferredContentSizeWhenEdgeAttached: Bool` **[NEW]** — width from `preferredContentSize` in landscape
- `UIViewController.sheetPresentationController: UISheetPresentationController?` **[NEW]** — access the sheet controller before presentation
- `UIPopoverPresentationController.adaptiveSheetPresentationController: UISheetPresentationController` **[NEW]** — configure the compact-class adaptive sheet from a popover

## Code Highlights

Medium + large detents with no-scroll-expand and undimmed medium:
```swift
func showImagePicker() {
    let picker = PHPickerViewController()
    picker.delegate = self
    if let sheet = picker.sheetPresentationController {
        sheet.detents = [.medium(), .large()]
        sheet.prefersScrollingExpandsWhenScrolledToEdge = false
        sheet.smallestUndimmedDetentIdentifier = .medium
    }
    present(picker, animated: true)
}
```

Animated detent selection on photo pick:
```swift
func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
    // apply result to image view
    if let sheet = picker.sheetPresentationController {
        sheet.animateChanges {
            sheet.selectedDetentIdentifier = .medium
        }
    }
}
```

Popover adapting to customized sheet on iPad:
```swift
func showImagePicker(_ sender: UIBarButtonItem) {
    let picker = PHPickerViewController()
    picker.delegate = self
    picker.modalPresentationStyle = .popover
    if let popover = picker.popoverPresentationController {
        popover.barButtonItem = sender
        let sheet = popover.adaptiveSheetPresentationController
        sheet.detents = [.medium(), .large()]
        sheet.prefersScrollingExpandsWhenScrolledToEdge = false
        sheet.smallestUndimmedDetentIdentifier = .medium
    }
    present(picker, animated: true)
}

// In delegate callback, access sheet consistently:
func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
    if let sheet = picker.popoverPresentationController?.adaptiveSheetPresentationController {
        sheet.animateChanges {
            sheet.selectedDetentIdentifier = .medium
        }
    }
}
```

Landscape edge-attached appearance with grabber:
```swift
if let sheet = fontPicker.sheetPresentationController {
    sheet.prefersEdgeAttachedInCompactHeight = true
    sheet.widthFollowsPreferredContentSizeWhenEdgeAttached = true
    sheet.prefersGrabberVisible = true
    sheet.preferredCornerRadius = 20.0
}
present(fontPicker, animated: true)
```

## Takeaways
- `UISheetPresentationController.detents = [.medium(), .large()]` is all that is required to get a half-height, resizable sheet — no custom container, no custom `UIPresentationController` subclass.
- Set `smallestUndimmedDetentIdentifier = .medium` to remove dimming at medium height, enabling genuine non-modal interactions with content behind the sheet.
- Wrap `selectedDetentIdentifier` changes in `animateChanges { }` to get smooth system-standard animations; omitting this causes instant, non-animated jumps.
- For popover-based flows, use `popoverPresentationController.adaptiveSheetPresentationController` to configure the compact-class sheet, and always read back through `popoverPresentationController?.adaptiveSheetPresentationController` in delegate callbacks.
- Automatic keyboard avoidance works with medium-height sheets — the sheet expands when the keyboard appears and collapses when it dismisses, with no extra code.

---
_Source: WWDC21 Session 10063 page (abstract, chapter summaries, code samples, and resource links)._
