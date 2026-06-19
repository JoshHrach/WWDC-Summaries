# Handle Trackpad and Mouse Input
**WWDC20 · Session 10094** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10094/)

_Platforms:_ iPadOS 14, macOS Big Sur 11 (Mac Catalyst)

## Overview
This session covers the full range of UIKit APIs for handling indirect pointer input — trackpads and mice — in iPad and Mac Catalyst apps. It is organized into two sections: common updates every app should adopt (hover detection, pointer locking, scroll input, trackpad pinch/rotate gestures) and advanced updates for pro user experiences (button mask, modifier flags, custom gesture event filtering, and indirect pointer touch types).

The session also explains the `UIApplicationSupportsIndirectInputEvents` Info.plist key, which moves apps out of a compatibility mode to gain the `TouchType.indirectPointer` and `EventType.transform` event types — a transition all apps built against the iOS 14 / Big Sur SDKs are expected to make.

## Key Topics

**UIHoverGestureRecognizer**
- Available on iPadOS 14 (previously Mac Catalyst only)
- `began` = pointer enters view bounds; `ended` = pointer exits view bounds
- Use for hiding/revealing UI content based on pointer position
- Do NOT use for pointer appearance changes — use `UIPointerInteraction` for that
- Driven by `UIEvent.EventType.hover` when inspecting via `UIApplication.sendEvent`

**Pointer Locking (New in iPadOS 14 / Mac Catalyst Big Sur)**
- `UIViewController.prefersPointerLocked: Bool` **[NEW]** — override to request pointer lock
- `UIViewController.setNeedsUpdateOfPrefersPointerLocked()` **[NEW]** — invalidates the cached value
- `UIPointerLockState` **[NEW]** — system-resolved lock state for the scene
  - `.isLocked: Bool` — whether pointer is actually locked
  - Observe `UIPointerLockState.didChangeNotification` for changes
- System evaluates lock eligibility continuously; no need to re-request on condition changes
- iPadOS requirements: scene must be full-screen (no Split View/Slide Over) and in `foregroundActive` state
- macOS requirements: app must be frontmost; window must be ordered to front
- `UIScene.pointerLockState` returns `nil` on scenes that do not support locking

**Scroll Input for Pan Gestures**
- `UIPanGestureRecognizer.allowedScrollTypesMask: UIScrollTypeMask` **[NEW]** — enables scroll event handling for the recognizer
- Default: empty mask (no scroll input); `UIScrollView`'s pan gesture handles all types automatically
- `UIScrollTypeMask.continuous` — trackpad two-finger scroll only
- `UIScrollTypeMask.discrete` — scroll wheel clicks only
- `UIScrollTypeMask.all` — both continuous and discrete
- Must be set on all custom pan gestures that should respond to trackpad/mouse scroll

**Trackpad Pinch and Rotate**
- `UIPinchGestureRecognizer` and `UIRotationGestureRecognizer` work automatically with trackpad
- Default: compatibility mode using gesture-simulating touches
- With `UIApplicationSupportsIndirectInputEvents = YES` in Info.plist: gestures receive `EventType.transform` directly from device — more precise, no incidental gesture activation
- When driven by `EventType.transform`: `numberOfTouches` returns 0; `locationOfTouch:inView:` may throw

**Button Mask (New)**
- `UIEvent.buttonMask: UIEvent.ButtonMask` **[NEW]** — set of buttons pressed during click
- `UIGestureRecognizer.buttonMask: UIEvent.ButtonMask` **[NEW]** — from last processed event (not current)
- `UITapGestureRecognizer.buttonMaskRequired: UIEvent.ButtonMask` **[NEW]** — require specific buttons before firing
- `UIEvent.ButtonMask.button(_:)` **[NEW]** — convenience for high-number mouse buttons
- Values: `.primary`, `.secondary`, `.button(3)`, `.button(4)`, etc.

**Modifier Flags (New)**
- `UIEvent.modifierFlags: UIKeyModifierFlags` **[NEW]** — keyboard modifiers at time of event
- `UIGestureRecognizer.modifierFlags: UIKeyModifierFlags` **[NEW]** — from last processed event
- Values: `.shift`, `.control`, `.alternate`, `.command`, `.numericPad`, `.alphaShift`

**Gesture Event Filtering (New)**
- `UIGestureRecognizer.shouldReceive(_ event: UIEvent) -> Bool` **[NEW]** — subclass override to filter by event type
- `UIGestureRecognizerDelegate.gestureRecognizer(_:shouldReceive:) -> Bool` **[NEW]** — delegate method for event filtering
- Both called before event processing — use `event.buttonMask` / `event.modifierFlags`, NOT the gesture's properties (not yet updated at this point)
- Called only for event types the gesture handles (pinch is not asked about scroll events)

**Indirect Pointer Touch Type**
- `UITouch.TouchType.indirectPointer` **[NEW]** — identifies touches from a pointing device vs. a finger
- Available only with `UIApplicationSupportsIndirectInputEvents = YES`
- Use with `UIGestureRecognizer.allowedTouchTypes` to target only pointer clicks or only finger touches
- Allows reducing expanded hit-test regions for the more precise pointer

**UIApplicationSupportsIndirectInputEvents (Info.plist key)**
- Not required for pointer interactions, clicks, scroll input, or trackpad gestures
- Required for `TouchType.indirectPointer` and `EventType.transform`
- Existing projects: add to Info.plist manually
- New UIKit/SwiftUI projects targeting iOS 14 / Big Sur SDKs: set to `YES` by default
- Future release: default will change; key will no longer be consulted

## APIs & Frameworks

### UIKit — Indirect Input
- `UIHoverGestureRecognizer` — hover detection (existing, now on iPadOS 14)
- `UIViewController.prefersPointerLocked: Bool` **[NEW]** — request pointer lock
- `UIViewController.setNeedsUpdateOfPrefersPointerLocked()` **[NEW]** — invalidate lock preference
- `UIScene.pointerLockState: UIPointerLockState?` **[NEW]** — access lock state for scene
- `UIPointerLockState.isLocked: Bool` **[NEW]** — resolved lock state
- `UIPointerLockState.didChangeNotification: Notification.Name` **[NEW]** — lock state change notification
- `UIPanGestureRecognizer.allowedScrollTypesMask: UIScrollTypeMask` **[NEW]** — enable scroll input
- `UIScrollTypeMask` **[NEW]** — `.continuous`, `.discrete`, `.all`
- `UIEvent.ButtonMask` **[NEW]** — button set type; `.primary`, `.secondary`, `.button(_:)`
- `UIEvent.buttonMask: UIEvent.ButtonMask` **[NEW]**
- `UIGestureRecognizer.buttonMask: UIEvent.ButtonMask` **[NEW]**
- `UITapGestureRecognizer.buttonMaskRequired: UIEvent.ButtonMask` **[NEW]**
- `UIEvent.modifierFlags: UIKeyModifierFlags` **[NEW]**
- `UIGestureRecognizer.modifierFlags: UIKeyModifierFlags` **[NEW]**
- `UIGestureRecognizer.shouldReceive(_ event: UIEvent) -> Bool` **[NEW]** — subclass override
- `UIGestureRecognizerDelegate.gestureRecognizer(_:shouldReceive event:) -> Bool` **[NEW]** — delegate override
- `UITouch.TouchType.indirectPointer` **[NEW]** — pointer click touch type
- `UIEvent.EventType.transform` **[NEW]** — trackpad pinch/rotate event type (with plist key)
- `UIEvent.EventType.scroll` — scroll wheel/trackpad scroll event type

## Code Highlights

Hover gesture for showing/hiding controls:
```swift
let controlsHover = UIHoverGestureRecognizer(target: self, action: #selector(handleHover))

@objc func handleHover(_ recognizer: UIHoverGestureRecognizer) {
    switch recognizer.state {
    case .began:  self.showsPlaybackControls = true
    case .ended:  self.showsPlaybackControls = false
    default: break
    }
}
```

Pointer locking in a game view controller:
```swift
class GameViewController: UIViewController {
    var shouldLockPointer = true
    override var prefersPointerLocked: Bool { shouldLockPointer }

    func disablePointerLock() {
        shouldLockPointer = false
        setNeedsUpdateOfPrefersPointerLocked()
    }
}

// Observing lock state changes:
if let lockState = scene.pointerLockState {
    NotificationCenter.default.addObserver(forName: UIPointerLockState.didChangeNotification,
                                           object: lockState, queue: .main) { note in
        let isLocked = (note.object as! UIPointerLockState).isLocked
        gameEngine.updatePointerLock(isLocked)
    }
}
```

Enabling scroll input for custom pan gestures:
```swift
drawerPan.allowedScrollTypesMask = [.continuous]       // trackpad two-finger only
pullToRefreshPan.allowedScrollTypesMask = [.all]        // trackpad + scroll wheel
```

Requiring a third mouse button:
```swift
self.thirdMouseButtonTap.buttonMaskRequired = .button(3)
```

Custom gesture that only handles secondary or Control-primary clicks:
```swift
class SecondaryClickGesture: UIGestureRecognizer {
    override func shouldReceive(_ event: UIEvent) -> Bool {
        let secondaryClick = event.buttonMask == .secondary
        let controlClick = event.buttonMask == .primary && event.modifierFlags == .control
        return secondaryClick || controlClick
    }
}
```

Hover gesture filtered by modifier key:
```swift
func gestureRecognizer(_ gr: UIGestureRecognizer, shouldReceive event: UIEvent) -> Bool {
    if gr == closedCaptionHover {
        return event.modifierFlags.contains(.alternate)
    }
    return true
}
```

## Takeaways
- Set `allowedScrollTypesMask` on every custom `UIPanGestureRecognizer` that should respond to trackpad/mouse scroll — `UIScrollView`'s pan gesture already handles this, but custom pans do not by default.
- Add the `UIApplicationSupportsIndirectInputEvents = YES` Info.plist key to exit compatibility mode and gain precise `EventType.transform` for pinch/rotate and the `TouchType.indirectPointer` type — all new projects targeting iOS 14 / Big Sur include this by default.
- In `shouldReceive(_:)` and its delegate equivalent, always inspect `event.buttonMask` and `event.modifierFlags`, not the properties on `UIGestureRecognizer` — the gesture's properties are not yet updated at that point in the call chain.
- Pointer locking is governed by the system — override `prefersPointerLocked` and observe `UIPointerLockState.didChangeNotification` rather than managing lock state manually.

---
_Source: WWDC20 Session 10094 page (abstract, transcript, code samples, and resource links)._
