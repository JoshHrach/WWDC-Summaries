# App Accessibility for Switch Control
**WWDC20 · Session 10019** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10019/)

_Platforms:_ iOS 14, iPadOS 14

## Overview
Switch Control is an assistive technology on Apple platforms that lets users with very limited motor abilities interact with their devices using one or more external switches or buttons — often mounted on wheelchairs and operated by head taps, tongue clicks, or breath control. This session explains how Switch Control works (including autoscanning mode), what makes apps difficult for Switch Control users (mistap risk, inefficient grouping, gestures requiring menu navigation), and which APIs and design approaches address these challenges.

A key message: if an app is 100% accessible for VoiceOver, it will likely already work well for Switch Control. A few additional steps — proper element grouping, explicit element ordering, follow-focus callbacks, and custom actions — can significantly improve the experience for Switch Control users. The session also introduces a new iOS 14 API allowing custom actions to display an image (using SF Symbols) instead of a first-letter placeholder.

## Key Topics
- **Switch Control basics** — External switch or button input; autoscanning (cursor advances automatically until the user activates a switch) vs. manual scanning (user advances cursor with one switch, activates with another); cursor navigates groups before individual items.
- **Key usability challenges** — Timing sensitivity (mistaps are costly), grouping efficiency (poor groups mean long wait times), error tolerance (accidental taps on destructive actions).
- **`accessibilityNavigationStyle = .combined`** — Set on a container view to tell Switch Control to treat the container's children as a single combined group; cursor enters the group together rather than scanning each child individually at the top level.
- **`accessibilityElements` array** — Explicitly ordering accessibility elements overrides default top-to-bottom, left-to-right navigation order; essential for non-linear layouts like a curvy game path.
- **Follow-focus callbacks** — `accessibilityElementDidBecomeFocused()` and `accessibilityElementDidLoseFocus()` on `UIView` subclasses; allow the app to respond to Switch Control (and VoiceOver) cursor movement — e.g., auto-flip a card when the cursor lands on it.
- **`UIAccessibilityCustomAction`** **with image** **[NEW in iOS 14]** — Custom actions surface additional behaviors (pin, add, delete, etc.) directly in the Switch Control top-level menu, avoiding the gestures submenu. New `image` property on `UIAccessibilityCustomAction` lets an SF Symbol appear in the menu instead of the default first-letter placeholder.
- **`UIAccessibility.isSwitchControlRunning`** — Static property to detect whether Switch Control is active; companion `UIAccessibility.switchControlStatusDidChangeNotification` notification.
- **`accessibilityRespondsToUserInteraction`** — Set to `true` on otherwise static views (e.g., `UILabel` with tap gesture) to make Switch Control include them in cursor navigation.
- **Best practices** — Confirm destructive actions; avoid session timeouts (Switch Control users enter info more slowly); group view hierarchy clearly; don't leave sensitive info on screen unnecessarily (Switch Control devices often mounted in public).

## APIs & Frameworks

### UIKit / Accessibility
- **`UIView.accessibilityNavigationStyle`** — `UIAccessibilityNavigationStyle`; `.automatic`, `.separate`, `.combined`; set `.combined` on a container to group children for Switch Control
- **`UIView.accessibilityElements`** — `[Any]?`; explicit ordered array of accessibility elements; overrides default navigation order
- **`UIView.accessibilityElementDidBecomeFocused()`** — Override in `UIView` subclass; called when VoiceOver/Switch Control cursor moves to this element
- **`UIView.accessibilityElementDidLoseFocus()`** — Override in `UIView` subclass; called when cursor leaves this element
- **`UIAccessibilityCustomAction`** — `init(name:handler:)` or `init(name:target:selector:)`
- **`UIAccessibilityCustomAction.image`** **[NEW]** — `UIImage?`; SF Symbol or custom image displayed in the Switch Control / VoiceOver action menu
- **`UIView.accessibilityCustomActions`** — `[UIAccessibilityCustomAction]?`; array of custom actions for the element
- **`UIAccessibility.isSwitchControlRunning`** — `static var Bool` (class property on `UIAccessibility`)
- **`UIAccessibility.switchControlStatusDidChangeNotification`** — `NSNotification.Name`
- **`UIView.accessibilityRespondsToUserInteraction`** — `Bool`; `true` makes Switch Control include static-appearing views in navigation

## Code Highlights

Group container children and set explicit ordering:
```swift
containerView.accessibilityNavigationStyle = .combined
containerView.accessibilityElements = [levelFourView, levelFiveView, levelSixView]
```

Auto-flip a card when Switch Control cursor arrives:
```swift
override func accessibilityElementDidBecomeFocused() {
    self.flip(to: .front)
}
override func accessibilityElementDidLoseFocus() {
    self.flip(to: .back)
}
```

Custom actions with SF Symbol images (iOS 14):
```swift
let pinAction = UIAccessibilityCustomAction(name: "Pin Card") { _ in
    self.setPinned(true); return true
}
pinAction.image = UIImage(systemName: "pin")  // NEW in iOS 14

let addAction = UIAccessibilityCustomAction(name: "Add Card") { _ in
    self.setSelected(true); return true
}
addAction.image = UIImage(systemName: "add.square")

self.accessibilityCustomActions = [addAction, pinAction]
```

## Takeaways
- A well-accessible VoiceOver app is usually already good for Switch Control; targeted improvements (grouping, ordering, follow-focus, custom actions) complete the experience.
- Use `accessibilityNavigationStyle = .combined` on container views and explicit `accessibilityElements` arrays to control Switch Control cursor grouping and traversal order for non-linear layouts.
- Custom actions (via `UIAccessibilityCustomAction`) surface frequently-needed behaviors in the Switch Control top-level menu, eliminating the need to navigate into the gestures submenu; iOS 14 adds `image` support for clearer menu icons.
- Avoid timeouts, require confirmation for destructive actions, and keep sensitive information off-screen — these practices disproportionately benefit Switch Control users who interact more slowly.

---
_Source: WWDC20 Session 10019 page (abstract, chapter summaries, code samples, and resource links)._
