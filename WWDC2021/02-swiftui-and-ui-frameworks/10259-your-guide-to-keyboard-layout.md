# Your Guide to Keyboard Layout
**WWDC21 · Session 10259** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10259/)

_Platforms:_ iOS 15, iPadOS 15

## Overview

iOS 15 introduces `UIKeyboardLayoutGuide`, a new Auto Layout layout guide that represents the space the on-screen keyboard occupies — eliminating the need to register for keyboard notifications, calculate frame offsets, and manually animate layout changes. A single constraint to the guide's `topAnchor` replaces dozens of lines of notification-observer code. The guide automatically matches all keyboard animations (show, hide, height changes), accounts for safe-area insets, and handles undocked and floating keyboards.

`UIKeyboardLayoutGuide` is a subclass of the also-new `UITrackingLayoutGuide`, which adds the ability to activate and deactivate constraint arrays based on how close the guide is to each screen edge. By combining `setConstraints(_:activeWhenNearEdge:)` and `setConstraints(_:activeWhenAwayFrom:)`, developers can build layouts that intelligently rearrange themselves as a floating keyboard moves around the screen — attaching UI controls to the keyboard when it is centered, and dropping them to the safe area when the keyboard drifts near the top of the screen.

## Key Topics

### From Notifications to a Layout Guide
Before iOS 15, keyboard handling required registering for `keyboardWillShowNotification` and related notifications, extracting frame and animation info from the `userInfo` dictionary, adjusting for the view's own coordinate space and safe-area insets, and running a `UIView.animate` block. That boilerplate collapses to a single Auto Layout constraint targeting `view.keyboardLayoutGuide.topAnchor`.

### UIKeyboardLayoutGuide Basics
- Accessed via `view.keyboardLayoutGuide` — a property on `UIView`, available in iOS 15+.
- Behaves like any `UILayoutGuide`: has `topAnchor`, `bottomAnchor`, `leadingAnchor`, `trailingAnchor`, `centerXAnchor`, `heightAnchor`, etc.
- Automatically animates with the keyboard; handles multiple keyboard heights (standard, emoji, QuickType suggestions, etc.).
- Accounts for safe-area insets automatically — no manual subtraction needed.
- When the keyboard is docked (default), the guide sits at the bottom of the window, full-width.
- When the keyboard is undocked or dismissed, the guide drops to the bottom of the screen by default.

### followsUndockedKeyboard
Setting `keyboardLayoutGuide.followsUndockedKeyboard = true` causes the guide to track the keyboard even when it is undocked, split, or floating. This enables UI that sticks to the floating keyboard as users drag it around the screen. When enabled, the keyboard leaving the app's bounds is treated as a dismiss.

### UITrackingLayoutGuide: Edge-Aware Constraints
`UITrackingLayoutGuide` (the superclass of `UIKeyboardLayoutGuide`) manages two sets of constraints per edge:
- `setConstraints(_:activeWhenNearEdge:)` — constraints activated when the guide approaches the specified edge, deactivated when it moves away.
- `setConstraints(_:activeWhenAwayFrom:)` — constraints activated when the guide is away from the specified edge.

Multiple edges can be specified (e.g., `[.leading, .trailing]`). The "near" region is determined by the system based on proximity to the edge; a floating keyboard can be near two adjacent edges simultaneously.

### Edge Semantics for Different Keyboard Types
- **Docked keyboard**: always near `.bottom`, away from all others.
- **Undocked/split keyboard**: away from all edges unless near `.top`.
- **Floating keyboard**: can be near or away from any edge, including two adjacent edges simultaneously.
- **Hardware keyboard adaptive shortcuts bar**: near `.bottom`; can be near `.leading` or `.trailing` when collapsed.
- **Camera keyboard**: behaves like a docked keyboard but may be nearly full-screen.

### Multitasking and Split-Screen Considerations
When another app shares the screen, the layout guide is sized to match only the portion of the keyboard overlapping the app's window. At the narrowest split sizes, the guide is always treated as away from horizontal edges (same as iPhone or a docked keyboard), while vertical proximity to `.top` is still tracked.

## APIs & Frameworks

- `UIKeyboardLayoutGuide` **[NEW]** — Auto Layout guide representing the on-screen keyboard area; property `view.keyboardLayoutGuide`
- `UITrackingLayoutGuide` **[NEW]** — layout guide base class that activates/deactivates constraint arrays by edge proximity
- `UITrackingLayoutGuide.setConstraints(_:activeWhenNearEdge:)` **[NEW]** — activates constraints when guide nears specified `NSDirectionalRectEdge`
- `UITrackingLayoutGuide.setConstraints(_:activeWhenAwayFrom:)` **[NEW]** — activates constraints when guide is away from specified edge(s)
- `UIKeyboardLayoutGuide.followsUndockedKeyboard: Bool` **[NEW]** — when `true`, guide tracks undocked/floating keyboards; default `false`
- `NSDirectionalRectEdge` — `.top`, `.leading`, `.trailing`, `.bottom` used for edge-proximity APIs
- `UILayoutGuide` — base class; `topAnchor`, `bottomAnchor`, `leadingAnchor`, `trailingAnchor`, `centerXAnchor`, etc.
- `UIResponder.keyboardWillShowNotification` / `keyboardWillHideNotification` — legacy notification approach (not deprecated, still available)

## Code Highlights

Minimal keyboard avoidance with the new guide (replaces notification boilerplate):
```swift
// One line replaces ~30 lines of notification code
view.keyboardLayoutGuide.topAnchor.constraint(
    equalToSystemSpacingBelow: textView.bottomAnchor,
    multiplier: 1.0
).isActive = true
```

Vertical adaptive layout — control follows keyboard unless keyboard is near the top:
```swift
let awayFromTopConstraints = [
    view.keyboardLayoutGuide.topAnchor.constraint(equalTo: editView.bottomAnchor),
]
view.keyboardLayoutGuide.setConstraints(awayFromTopConstraints, activeWhenAwayFrom: .top)

let nearTopConstraints = [
    view.safeAreaLayoutGuide.bottomAnchor.constraint(equalTo: editView.bottomAnchor),
]
view.keyboardLayoutGuide.setConstraints(nearTopConstraints, activeWhenNearEdge: .top)
```

Horizontal adaptive layout — control centers on keyboard when floating, snaps to edge when near it:
```swift
let awayFromSides = [
    view.keyboardLayoutGuide.centerXAnchor.constraint(equalTo: editView.centerXAnchor),
    imageView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
]
view.keyboardLayoutGuide.setConstraints(awayFromSides, activeWhenAwayFrom: [.leading, .trailing])

let nearTrailingConstraints = [
    view.keyboardLayoutGuide.trailingAnchor.constraint(equalTo: editView.trailingAnchor),
    imageView.leadingAnchor.constraint(
        equalToSystemSpacingAfter: view.safeAreaLayoutGuide.leadingAnchor, multiplier: 1.0),
]
view.keyboardLayoutGuide.setConstraints(nearTrailingConstraints, activeWhenNearEdge: .trailing)
```

## Takeaways

- `UIKeyboardLayoutGuide` eliminates all notification-based keyboard layout code for the common case — a single topAnchor constraint is all that is needed for most apps.
- `followsUndockedKeyboard = true` combined with `UITrackingLayoutGuide` edge constraints enables rich floating-keyboard experiences, but requires careful handling of all edge states (floating away from everything, near top, near corners) to avoid broken layouts.
- The guide is sized to match only the keyboard area over the app window in multitasking scenarios, so layouts built with it behave correctly in Split View and Slide Over automatically.
- Stop thinking of the keyboard as something to fight or avoid — with `UIKeyboardLayoutGuide`, it is a first-class participant in Auto Layout, no different from a safe-area guide or a view's bounds.

---
_Source: WWDC21 Session 10259 page (abstract, transcript, and code samples)._
