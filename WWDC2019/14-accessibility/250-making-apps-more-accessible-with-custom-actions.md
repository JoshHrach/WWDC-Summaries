# Making Apps More Accessible With Custom Actions
**WWDC19 · Session 250** · [Watch](https://developer.apple.com/videos/play/wwdc2019/250/)

_Platforms:_ iOS 13, iPadOS 13

## Overview
Accessibility custom actions let you attach named operations directly to a view, allowing assistive technologies to surface them without cluttering the on-screen element hierarchy. This session explains how custom actions benefit VoiceOver users (reducing swipe counts and cognitive load) and Switch Control users (cutting required taps from 18 down to 6 in a real demo), and introduces two new technologies in iOS 13 that also consume custom actions: Full Keyboard Access and Voice Control.

The core concept is simple: instead of exposing many small, repetitive buttons as separate accessibility elements in every list row, you mark those buttons as non-accessible and attach their behaviors as named actions on the cell itself. The user then picks the desired action via a swipe (VoiceOver) or the Switch Control menu, keeping navigation fast and unambiguous.

iOS 13 adds a block-based initializer for `UIAccessibilityCustomAction`, making it easier to define actions inline without needing a separate target/selector pair.

## Key Topics
- **Reducing clutter for VoiceOver** — make per-row action buttons non-accessible (`isAccessibilityElement = false`) and express them as custom actions on the containing cell; users swipe vertically to browse actions and double-tap to invoke
- **Improving Switch Control efficiency** — custom actions appear on page 1 of the Switch Control Menu, eliminating deep gesture-menu navigation; the demo went from 18 taps to 6 (67% reduction)
- **Full Keyboard Access support** — custom actions are now surfaced to users navigating with a hardware keyboard **[NEW iOS 13]**
- **Voice Control support** — custom actions are now accessible to Voice Control users **[NEW iOS 13]**
- **Block-based custom action initializer** — new `UIAccessibilityCustomAction(name:actionHandler:)` **[NEW iOS 13]**

## APIs & Frameworks
- **UIKit / Accessibility**
  - `UIAccessibilityCustomAction` — represents a named action for assistive technologies
    - `init(name:target:selector:)` — existing initializer
    - `init(name:actionHandler:)` **[NEW iOS 13]** — block-based initializer
    - `init(name:image:target:selector:)` — with icon image
    - `init(name:image:actionHandler:)` **[NEW iOS 13]** — block-based with image
  - `UIAccessibility.accessibilityCustomActions: [UIAccessibilityCustomAction]?` — property to override on any `UIView`/`UIAccessibilityElement`
  - `UIView.isAccessibilityElement: Bool` — set `false` on action buttons that are now covered by custom actions on the parent cell
  - VoiceOver — surfaces custom actions; user swipes up/down with one finger to browse, double-taps to invoke
  - Switch Control — shows custom actions on page 1 of the Switch Control Menu
  - Full Keyboard Access **[NEW iOS 13]** — consumes custom actions
  - Voice Control **[NEW iOS 13]** — consumes custom actions

## Code Highlights

```swift
// Existing target/selector style
override var accessibilityCustomActions: [UIAccessibilityCustomAction]? {
    get {
        return [
            UIAccessibilityCustomAction(
                name: "Toggle Favorite",
                target: self,
                selector: #selector(toggleFavorite(_:))
            ),
            UIAccessibilityCustomAction(
                name: "Increase Rating",
                target: self,
                selector: #selector(increaseRating(_:))
            )
        ]
    }
    set {}
}

@objc func increaseRating(_ action: UIAccessibilityCustomAction) -> Bool {
    rating += 1
    return true
}
```

```swift
// iOS 13: block-based initializer
override var accessibilityCustomActions: [UIAccessibilityCustomAction]? {
    get {
        return [
            UIAccessibilityCustomAction(name: "Increase Rating") { [weak self] _ in
                self?.rating += 1
                return true
            }
        ]
    }
    set {}
}
```

```swift
// Hide repetitive sub-buttons from assistive technologies
increaseRatingButton.isAccessibilityElement = false
decreaseRatingButton.isAccessibilityElement = false
favoriteButton.isAccessibilityElement = false
```

## Takeaways
- Custom actions are the primary tool for reducing repetitive accessibility elements in lists and collection views — hide the individual buttons and attach their behaviors to the parent cell.
- They work across VoiceOver, Switch Control, Full Keyboard Access, and Voice Control simultaneously from a single implementation point.
- The new iOS 13 block-based initializer (`actionHandler:`) makes inline action definition cleaner, avoiding separate target/selector pairs.
- Turn on VoiceOver, Switch Control, Full Keyboard Access, and Voice Control yourself when reviewing your app; the interaction overhead immediately reveals where custom actions would help.

---
_Source: WWDC19 Session 250 page (abstract, chapter summaries, code samples, and resource links)._
