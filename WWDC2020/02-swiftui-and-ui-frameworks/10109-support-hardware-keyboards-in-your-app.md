# Support hardware keyboards in your app
**WWDC20 · Session 10109** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10109/)

_Platforms:_ iOS 14, iPadOS 14, Mac Catalyst

## Overview
iPadOS and Mac Catalyst apps used with hardware keyboards should support keyboard shortcuts, modifier-key interactions with gestures, and raw key-down/key-up event handling. This session walks through the UIKit responder chain model and covers four areas: implementing custom `UIKeyCommand` shortcuts, leveraging `UIResponderStandardEditActions` for common edit operations without writing any key commands, using modifier flags on gesture recognizers, and using the new `pressesBegan`/`pressesEnded` raw keyboard event API introduced in iOS 13.4.

The session uses music playback (spacebar to play/pause), file list multi-selection (Shift + tap / Command + tap), canvas drag constraints (Shift to constrain aspect ratio), and arrow-key continuous movement of canvas objects as concrete examples throughout.

## Key Topics

### The Responder Chain and `UIKeyCommand`
- `UIKeyCommand` is a `UICommand` subclass representing a custom keyboard shortcut — it carries a title (shown in discoverability HUD), input string, optional modifier flags, and an action selector
- Collect key commands per responder by overriding `var keyCommands: [UIKeyCommand]?` on any `UIResponder` subclass (views, view controllers)
- The system walks the responder chain from first responder to `UIApplication` collecting all key commands — commands from deeper in the chain take priority
- Hold the Command key anywhere in the system to display the **discoverability HUD** listing all active commands
- `UIKeyCommand` is a subclass of `UICommand` → integrates directly with the `UIMenuBuilder` API for macOS menu bar items in Mac Catalyst apps (no extra work for standard edit actions)
- Two prerequisites for a view controller to receive key commands: override `canBecomeFirstResponder` → return `true`; call `becomeFirstResponder()` in `viewDidAppear(_:)`

### `UIResponderStandardEditActions` (no key command boilerplate)
- `UIResponder` conforms to `UIResponderStandardEditActions` — a protocol with pre-defined methods covering common operations: `selectAll(_:)`, `copy(_:)`, `paste(_:)`, `cut(_:)`, `delete(_:)`, `find(_:)`, and more
- Override only the relevant methods on your view or view controller; the system automatically maps these to the expected keyboard shortcuts (e.g., Command-A → `selectAll`, Command-C → `copy`)
- Mac Catalyst: these actions are wired to the macOS menu bar automatically — no `UIMenuBuilder` code needed

### Multi-Selection with Modifier Keys in Table/Collection Views
- Override `tableView(_:shouldBeginMultipleSelectionInteractionAt:)` → return `true` to enable Shift-tap contiguous selection and Command-tap toggle selection without entering edit mode
- The system automatically enters editing/multiple-selection mode when a modifier key is held
- Override `tableView(_:didBeginMultipleSelectionInteractionAt:)` to update surrounding UI (toolbar buttons, etc.) when editing mode is triggered by a keyboard shortcut

### Modifier Flags on Gesture Recognizers (iOS 13.4+)
- `UIGestureRecognizer.modifierFlags` — a `UIKeyModifierFlags` value set to the modifier keys held when the gesture recognizer's state changed
- Works with any gesture recognizer (pan, tap, pinch, etc.) — check `.shift`, `.command`, `.alternate`, `.control` flags in the gesture callback
- Enables behaviors like: Shift + pan to constrain aspect ratio; Command + tap to add to selection

### Raw Keyboard Events: `pressesBegan` / `pressesEnded` (NEW in iOS 13.4)
- `UIResponder.pressesBegan(_:with:)` — called each time a physical key goes down; `pressesEnded(_:with:)` — called when released
- `UIPress.key` — a `UIKey` object providing `keyCode` (a `UIKeyboardHIDUsage` enum), `modifierFlags`, and `characters`
- `UIKeyboardHIDUsage` cases: `.keyboardUpArrow`, `.keyboardDownArrow`, `.keyboardLeftArrow`, `.keyboardRightArrow`, `.keyboardSpacebar`, `.keyboardReturnOrEnter`, etc.
- Unlike `UIKeyCommand` (which fires once), `pressesBegan`/`pressesEnded` support continuous hold — ideal for continuous movement, held modifier detection, or game-like input
- Call `super.pressesBegan` for any keys not handled to propagate up the responder chain

## APIs & Frameworks

**UIKit — Keyboard Shortcuts**
- `UIKeyCommand` — keyboard shortcut object; subclass of `UICommand`
  - `init(title:image:action:input:modifierFlags:discoverabilityTitle:attributes:state:)` **[iOS 13]**
  - `UIKeyModifierFlags` — `.command`, `.shift`, `.alternate`, `.control`, `.numericPad`
- `UIResponder.keyCommands: [UIKeyCommand]?` — override to supply commands for this responder
- `UIResponder.canBecomeFirstResponder: Bool` — override, return `true`
- `UIResponder.becomeFirstResponder()` — call in `viewDidAppear`
- `UIResponder.next: UIResponder?` — responder chain traversal

**UIKit — Standard Edit Actions**
- `UIResponderStandardEditActions` protocol — `selectAll(_:)`, `copy(_:)`, `paste(_:)`, `cut(_:)`, `delete(_:)`, `find(_:)`, `findAndReplace(_:)`, `useSelectionForFind(_:)`, and more — override as needed

**UIKit — Multi-Selection**
- `UITableViewDelegate.tableView(_:shouldBeginMultipleSelectionInteractionAt:) -> Bool`
- `UITableViewDelegate.tableView(_:didBeginMultipleSelectionInteractionAt:)`
- `UICollectionViewDelegate` equivalents for collection views

**UIKit — Gesture Recognizer Modifier Flags (iOS 13.4+)**
- `UIGestureRecognizer.modifierFlags: UIKeyModifierFlags` — modifier keys active during the gesture

**UIKit — Raw Key Events (iOS 13.4+)**
- `UIResponder.pressesBegan(_: Set<UIPress>, with: UIPressesEvent?)` **[NEW]**
- `UIResponder.pressesEnded(_: Set<UIPress>, with: UIPressesEvent?)` **[NEW]**
- `UIPress.key: UIKey?` — the key that triggered the press
- `UIKey.keyCode: UIKeyboardHIDUsage` — HID usage code for the physical key
- `UIKey.modifierFlags: UIKeyModifierFlags`
- `UIKey.characters: String` — printable character representation
- `UIKeyboardHIDUsage` — enum with cases for every keyboard key

**Mac Catalyst**
- `UIMenuBuilder` — `buildMenu(with:)` on `UIApplicationDelegate` to add/modify macOS menu items
- `UICommand` — base class for menu items; `UIKeyCommand` subclasses it

## Code Highlights

Custom key command (spacebar to play/pause) in a view controller:
```swift
class PlayerViewController: UIViewController {
    override var canBecomeFirstResponder: Bool { true }
    override func viewDidAppear(_ animated: Bool) { becomeFirstResponder() }
    override var keyCommands: [UIKeyCommand]? {
        [UIKeyCommand(title: NSLocalizedString("PLAY_PAUSE", comment: ""),
                      action: #selector(playPause),
                      input: " ")]
    }
}
```

Standard edit actions — no UIKeyCommand needed:
```swift
class SongListTableViewController: UITableViewController {
    override var canBecomeFirstResponder: Bool { true }
    override func viewDidAppear(_ animated: Bool) { becomeFirstResponder() }
    override func selectAll(_ sender: Any?) { /* select all songs */ }
    override func copy(_ sender: Any?) { /* copy selected songs */ }
    override func paste(_ sender: Any?) { /* paste */ }
}
```

Modifier flags on a pan gesture recognizer:
```swift
func recognizedDragGesture(_ panGesture: UIPanGestureRecognizer) {
    if panGesture.modifierFlags.contains(.shift) {
        constrainAspectRatio = true
    } else if panGesture.modifierFlags.contains(.command) {
        snapToGrid = true
    }
}
```

Raw key events for continuous arrow-key movement:
```swift
class CanvasViewController: UIViewController {
    override func pressesBegan(_ presses: Set<UIPress>, with event: UIPressesEvent?) {
        for press in presses {
            guard let key = press.key else { continue }
            switch key.keyCode {
            case .keyboardUpArrow:    startMoveUp()
            case .keyboardDownArrow:  startMoveDown()
            case .keyboardLeftArrow:  startMoveLeft()
            case .keyboardRightArrow: startMoveRight()
            default: super.pressesBegan(presses, with: event)
            }
        }
    }
    override func pressesEnded(_ presses: Set<UIPress>, with event: UIPressesEvent?) {
        stopMoving()
    }
}
```

## Takeaways

- Override `keyCommands` on any `UIResponder` to add discoverable keyboard shortcuts; the discoverability HUD (hold Command) surfaces them to users and helps during development.
- Override `UIResponderStandardEditActions` methods (`selectAll`, `copy`, `paste`, etc.) instead of defining `UIKeyCommand` for standard edit operations — the system wires the correct shortcuts and, on Mac Catalyst, the macOS menu bar automatically.
- Use `UIGestureRecognizer.modifierFlags` to gate modifier-key behaviors (constrain aspect ratio, snap to grid) inside existing gesture callbacks without adding any new key commands.
- Use `pressesBegan`/`pressesEnded` for continuous hold behaviors (arrow-key movement, held-key detection) that fire once per state change rather than once per key repeat like `UIKeyCommand`.

---
_Source: WWDC20 Session 10109 page (transcript and code samples)._
