# Support Full Keyboard Access in Your iOS App
**WWDC21 · Session 10120** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10120/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
Full Keyboard Access, introduced in iOS 13.4.1, allows users with motor impairments to interact with their iOS and iPadOS devices entirely via an external keyboard — without ever touching the screen. It is a middle ground between AssistiveTouch and Switch Control, and provides an alternative to Voice Control for users who are nonverbal or in environments where voice input is impractical.

This session walks through key APIs and best practices using a real-world game app ("Shape Shuffle") as a case study. It covers custom actions, keyboard shortcuts, focus exclusion, user input labels for the Find feature, and accessibility paths for shaped focus rings.

The session also highlights that the underlying focus engine driving Full Keyboard Access is the same engine used for Tab navigation across iPadOS, tvOS, and other Apple platforms.

## Key Topics

### Motor Accessibility Overview
iOS supports multiple assistive technologies for motor impairments: AssistiveTouch (simplified gestures), Switch Control (external switches/buttons), Voice Control (voice input), and Full Keyboard Access (hardware keyboard). Each technology serves different levels of motor ability.

### Full Keyboard Access Navigation
Tab moves focus, Space activates, Shift-Tab moves backward, Tab-Z opens a contextual actions menu, Tab-F opens the Find overlay, and arrow keys + Space navigate menus. Command key hold reveals keyboard shortcut HUD. All commands are fully customizable by the user.

### Custom Accessibility Actions
`UIAccessibilityCustomAction` entries appear in the Full Keyboard Access Tab-Z actions menu, in Switch Control scanning menus, and in VoiceOver. This is the preferred cross-technology approach for exposing common actions without requiring mouse/touch.

### Custom Keyboard Shortcuts
`UIKeyCommand` entries registered via `buildMenu(with:)` in `AppDelegate` appear in the Command-key HUD. They work both for Full Keyboard Access users and for standard keyboard users. `canPerformAction(_:withSender:)` can conditionally hide/show commands based on app state.

### Focus Exclusion: `accessibilityRespondsToUserInteraction`
Setting this to `false` on an `isAccessibilityElement = true` view tells Full Keyboard Access and Switch Control to skip the element (VoiceOver still reads it). This replaces the anti-pattern of overriding `canBecomeFocused`, which affects the entire focus engine.

### User Input Labels for Find
`accessibilityUserInputLabels` accepts an array of synonyms/aliases for image-based controls. These labels are used by Full Keyboard Access Find (Tab-F search) and by Voice Control tap commands. They do not affect the VoiceOver `accessibilityLabel`.

### Accessibility Path for Shaped Focus Rings
`accessibilityPath` (a `UIBezierPath` in screen coordinates) controls the shape of the Full Keyboard Access focus ring, matching it to the visual shape of non-rectangular buttons. For scroll views, override `accessibilityPath` as a computed property to keep coordinates correct during scrolling.

## APIs & Frameworks

**UIKit — Accessibility**
- `UIAccessibilityCustomAction(name:image:handler:)` — adds actions to the Tab-Z contextual menu for Full Keyboard Access, Switch Control, and VoiceOver
- `UIView.accessibilityCustomActions: [UIAccessibilityCustomAction]` — array of custom actions on a view
- `UIView.isAccessibilityElement: Bool` — marks a view as an accessibility element
- `UIView.accessibilityLabel: String` — label read by VoiceOver
- `UIView.accessibilityRespondsToUserInteraction: Bool` **[NEW in iOS 15 context]** — when `false`, tells motor technologies (Full Keyboard Access, Switch Control) to skip this element
- `UIView.accessibilityUserInputLabels: [String]` — synonym labels for Full Keyboard Access Find and Voice Control
- `UIView.accessibilityPath: UIBezierPath?` — custom focus ring shape in screen coordinates
- `UIView.accessibilityFrame: CGRect` — custom accessible frame

**UIKit — Keyboard Shortcuts (new in iOS/iPadOS 15, previously Mac Catalyst 13)**
- `UIKeyCommand(title:image:action:input:discoverabilityTitle:)` **[NEW on iOS/iPadOS 15]** — defines a keyboard shortcut with a discoverable title for the Command HUD
- `UIMenuBuilder` — used in `AppDelegate.buildMenu(with:)` to register menus and commands
- `UIMenu(title:image:identifier:children:)` — groups `UIKeyCommand` entries into a named menu
- `UIMenuBuilder.insertSibling(_:afterMenu:)` — inserts a menu into the app's menu bar/command system
- `UIMenu.Identifier` — typed identifier for custom menus
- `UIResponder.canPerformAction(_:withSender:)` — conditionally enables/disables key commands based on app state

**UIBezierPath**
- `UIBezierPath(ovalIn:)` — circular path for shaped focus rings

## Code Highlights

Custom actions (appear in Tab-Z menu):
```swift
let addAction = UIAccessibilityCustomAction(
    name: gameLocString("add"), image: UIImage(systemName: "plus.square")) { _ in
        self.addCard(); return true
    }
cardView.accessibilityCustomActions = [addAction, pinAction]
```

Keyboard shortcut registration via `buildMenu`:
```swift
override func buildMenu(with builder: UIMenuBuilder) {
    super.buildMenu(with: builder)
    guard builder.system == .main else { return }
    let addCommand = UIKeyCommand(title: gameLocString("add"),
        image: UIImage(systemName: "plus.square"),
        action: #selector(GameViewController.addFocusedCard),
        input: "A",
        discoverabilityTitle: gameLocString("add.card"))
    let menu = UIMenu(title: gameLocString("gameplay"),
        identifier: UIMenu.Identifier("gameplay_menu"),
        children: [addCommand, pinCommand])
    builder.insertSibling(menu, afterMenu: .view)
}
```

Excluding non-interactive elements from motor navigation:
```swift
itemView.isAccessibilityElement = true
itemView.accessibilityLabel = gameLocString(for: item)
itemView.accessibilityRespondsToUserInteraction = false
```

User input labels for Find/Voice Control:
```swift
self.accessibilityUserInputLabels = [
    gameLocString("settings"), gameLocString("prefs"),
    gameLocString("preferences"), gameLocString("gear")]
```

Shaped focus ring (computed override for scroll views):
```swift
override var accessibilityPath: UIBezierPath? {
    get { UIBezierPath(ovalIn: self.convert(self.bounds, to: nil)) }
    set { }
}
```

## Takeaways
- Add `UIAccessibilityCustomAction` entries for all common actions — they work across Full Keyboard Access (Tab-Z menu), Switch Control, and VoiceOver simultaneously.
- Use `accessibilityRespondsToUserInteraction = false` (not `canBecomeFocused`) to prevent motor navigation from landing on non-interactive but VoiceOver-visible elements.
- Populate `accessibilityUserInputLabels` with synonyms on image-based controls so Full Keyboard Access Find and Voice Control can discover them reliably.
- `UIKeyCommand` + `buildMenu(with:)`, new on iOS/iPadOS 15, provides discoverable keyboard shortcuts that serve both Full Keyboard Access users and standard keyboard power users.

---
_Source: WWDC21 Session 10120 page (abstract, chapter summaries, code samples, and resource links)._
