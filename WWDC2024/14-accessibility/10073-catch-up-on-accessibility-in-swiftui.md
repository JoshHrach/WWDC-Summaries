# Catch Up on Accessibility in SwiftUI
**WWDC24 · Session 10073** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10073/)

_Platforms:_ iOS 18, iPadOS 18, macOS 15, tvOS 18, watchOS 11, visionOS 2

## Overview
This session surveys the accessibility improvements Apple added to SwiftUI across recent releases. Rather than focusing on a single new feature, it sweeps through multiple enhancements — from richer VoiceOver descriptions to improved Switch Control and Direct Touch support — giving developers a compact reference for what changed and how to adopt it.

The presentation is structured around practical scenarios: navigating a list, interacting with custom controls, and handling focus. For each scenario the speaker contrasts the old behavior with the new API and shows how a small modifier addition or change dramatically improves the assistive-technology experience with little effort.

## Key Topics
- **Accessibility label enhancements** — new modifiers for richer, multi-element descriptions that give VoiceOver enough context without requiring custom accessibility elements.
- **Direct Touch & Pass-Through** — configuring custom drawing canvases and game views to forward raw touch events to accessibility clients when needed.
- **Switch Control improvements** — ensuring custom controls are reachable and activatable with Switch Control scanning.
- **Focus management** — using `AccessibilityFocusState` to programmatically direct VoiceOver focus when views appear or after an action completes.
- **Accessibility notifications** — posting announcements and layout-changed signals from SwiftUI code.

## APIs & Frameworks

**SwiftUI (Accessibility modifiers)**
- `accessibilityLabel(_:)` — set the primary spoken label for a view
- `accessibilityValue(_:)` — communicate current value to assistive tech
- `accessibilityHint(_:)` — describe what happens when a user interacts
- `accessibilityAction(_:_:)` — add named custom actions
- `accessibilityElement(children:)` — control grouping; `.combine` collapses children into a single element
- `accessibilityChildren(children:)` — expose custom virtual children for complex views
- `accessibilityRepresentation(representation:)` — replace a view's accessibility representation with a simpler SwiftUI view
- **[NEW]** `accessibilityDirectTouch(isEnabled:options:)` — opt a region into direct (pass-through) touch delivery for VoiceOver users
- **[NEW]** `AccessibilityDirectTouchOptions` — `.silentOnTouch` (suppress sound), `.requiresActivation` (need double-tap first)
- `accessibilityActivationPoint(_:)` — override the point VoiceOver uses to activate the element
- `AccessibilityFocusState` — property wrapper; bind to a `Bool` or a hashable enum to drive VoiceOver focus
- `accessibilityFocused(_:)` / `accessibilityFocused(_:equals:)` — connect a view to an `AccessibilityFocusState`
- `AccessibilityNotification` — namespace for posting notifications
  - `AccessibilityNotification.Announcement` — post a spoken announcement
  - `AccessibilityNotification.LayoutChanged` — signal a layout change to reorient VoiceOver
  - `AccessibilityNotification.ScreenChanged` — signal a full screen replacement
- `accessibilitySortPriority(_:)` — control VoiceOver navigation order
- `accessibilityHidden(_:)` — hide decorative views from assistive tech
- `accessibilityInputLabels(_:)` — alternate labels for Voice Control commands

## Code Highlights
Post a layout change notification after inserting new list content:

```swift
Button("Load More") {
    items.append(contentsOf: newItems)
    AccessibilityNotification.LayoutChanged().post()
}
```

Enable direct touch on a drawing canvas with silent pass-through:

```swift
Canvas { context, size in
    // drawing code
}
.accessibilityDirectTouch(isEnabled: true, options: .silentOnTouch)
```

Drive focus to a newly presented alert:

```swift
@AccessibilityFocusState var isFocused: Bool

VStack { … }
.onAppear { isFocused = true }
.accessibilityFocused($isFocused)
```

## Takeaways
- Use `.accessibilityRepresentation` to swap a complex custom view with a semantically equivalent native SwiftUI control — zero custom accessibility element code needed.
- Prefer `.accessibilityElement(children: .combine)` to collapse decorative sub-views into a single VoiceOver stop rather than building manual `accessibilityLabel` strings.
- Adopt `accessibilityDirectTouch` for drawing and game surfaces so VoiceOver users can interact naturally without being trapped in a black box.
- Post `AccessibilityNotification.LayoutChanged()` after dynamic data loads to keep VoiceOver's virtual cursor synchronized.

---
_Source: WWDC24 Session 10073 page (abstract, chapter summaries, code samples, and resource links)._
