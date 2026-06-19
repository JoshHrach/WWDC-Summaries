# Bring Keyboard and Mouse Gaming to iPad
**WWDC20 · Session 10617** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10617/)

_Platforms:_ iPadOS 14, macOS Big Sur 11

## Overview
iPadOS 14 adds game-focused keyboard and mouse APIs to the Game Controller framework via two new classes: `GCKeyboard` and `GCMouse`. These complement the existing UIKit pointer and keyboard APIs and are specifically designed for full-screen games that need raw delta mouse input, custom pointer rendering, input polling in render loops, and suppression of system gestures (Dock, Control Center, multitasking). A companion UIKit API (`prefersPointerLocked`) lets a `UIViewController` hide the system pointer and lock it to the app.

The session walks through adding keyboard (WASD + modifiers) and mouse (delta-based camera, scroll wheel zoom, left-button attack) to the Fox2 sample game and explains the decision tree for when to use `GCKeyboard`/`GCMouse` versus the UIKit pointer APIs introduced in iOS 13.4.

## Key Topics
- **`GCKeyboard`** **[NEW]** — Represents coalesced keyboard input from all connected keyboards; accessed via `GCKeyboard.coalesced`; conforms to `GCDevice`; exposes `keyboardInput` (`GCKeyboardInput`) with per-key button objects keyed by `GCKeyCode`.
- **`GCMouse`** **[NEW]** — Represents a connected pointing device; accessed via `GCMouse.currentMouse`; exposes `mouseInput` (`GCMouseInput`) with axes for mouse deltas and buttons for clicks; delivers raw delta events (not screen coordinates).
- **`GCKeyCode`** **[NEW]** — Enum of USB HID keyboard usage codes; used to address specific keys: `.keyW`, `.keyA`, `.keyS`, `.keyD`, `.spacebar`, `.leftShift`, etc.
- **Keyboard input modes** — Change handlers: `keyboardInput.keyChangedHandler` (any key) or `keyboardInput.button(forKeyCode:)?.valueChangedHandler` (specific key). Polling: `button(forKeyCode:)?.isPressed` — O(1), non-blocking, thread-safe; safe to call from render/simulation threads.
- **Mouse delta input** — `mouseInput.mouseMovedHandler` delivers `(mouse, deltaX, deltaY)` — raw device delta since last event, not screen position. App is responsible for implementing cursor position, acceleration, and edge behavior.
- **Pointer locking** — `UIViewController.prefersPointerLocked` **[NEW]** — override to return `true` to hide the system pointer and prevent it from triggering system gestures (Dock, Control Center, multitasking edges); call `setNeedsUpdateOfPrefersPointerLocked()` to dynamically change the value.
- **Device discovery** — Same pattern as `GCController`: observe `GCMouseDidConnectNotification` / `GCMouseDidDisconnectNotification` and `GCKeyboardDidConnectNotification` / `GCKeyboardDidDisconnectNotification`.
- **GCKeyboard vs UIKit** — Use `GCKeyboard`/`GCMouse` when: full-screen, custom pointer or no pointer, custom input acceleration, need polling in non-main threads, system gestures interfere. Use UIKit APIs when: optionally full-screen, standard pointer morphing fits, UIResponder callbacks on main thread are adequate.

## APIs & Frameworks

### Game Controller (GameController.framework)
- **`GCKeyboard`** **[NEW]** — `class GCKeyboard: NSObject, GCDevice`
  - `class var coalesced: GCKeyboard?` — Returns the merged input from all connected keyboards
  - `var keyboardInput: GCKeyboardInput?`
- **`GCKeyboardInput`** **[NEW]** — `GCPhysicalInputProfile` subclass
  - `var keyChangedHandler: ((GCKeyboardInput, GCControllerButtonInput, GCKeyCode, Bool) -> Void)?`
  - `func button(forKeyCode: GCKeyCode) -> GCControllerButtonInput?`
- **`GCKeyCode`** **[NEW]** — `struct GCKeyCode: RawRepresentable`; constants: `.keyA`…`.keyZ`, `.digit0`…`.digit9`, `.spacebar`, `.returnOrEnter`, `.leftShift`, `.rightShift`, `.leftControl`, `.rightControl`, `.leftAlt`, `.rightAlt`, `.leftCommand`, `.rightCommand`, `.upArrow`, `.downArrow`, `.leftArrow`, `.rightArrow`, `.escape`, `.tab`, and all standard HID keyboard keys
- **`GCMouse`** **[NEW]** — `class GCMouse: NSObject, GCDevice`
  - `class var current: GCMouse?`
  - `var mouseInput: GCMouseInput?`
- **`GCMouseInput`** **[NEW]** — `GCPhysicalInputProfile` subclass
  - `var mouseMovedHandler: ((GCMouseInput, Float, Float) -> Void)?` — `(mouse, deltaX, deltaY)`
  - `var leftButton: GCControllerButtonInput`
  - `var rightButton: GCControllerButtonInput?`
  - `var middleButton: GCControllerButtonInput?`
  - `var auxiliaryButtons: [GCControllerButtonInput]?`
  - `var scroll: GCControllerDirectionPad` — scroll wheel axes
- **Notifications** **[NEW]**:
  - `GCKeyboardDidConnectNotification`
  - `GCKeyboardDidDisconnectNotification`
  - `GCMouseDidConnectNotification`
  - `GCMouseDidDisconnectNotification`

### UIKit
- **`UIViewController.prefersPointerLocked`** **[NEW]** — `var prefersPointerLocked: Bool { get }` — override to return `true` to hide system pointer and prevent system gesture triggers
- **`UIViewController.setNeedsUpdateOfPrefersPointerLocked()`** **[NEW]** — Invalidates `prefersPointerLocked` so the system re-queries it

## Code Highlights

Keyboard event handler and polling in the game loop:
```swift
// Change handler for any key
if let keyboard = GCKeyboard.coalesced?.keyboardInput {
    keyboard.keyChangedHandler = { (keyboard, key, keyCode, pressed) in
        // handle key event
    }
    keyboard.button(forKeyCode: .spacebar)?.valueChangedHandler = { (key, value, pressed) in
        if pressed { character.jump() }
    }
}

// Polling in the render/update loop (thread-safe, O(1))
func pollInput() {
    if let keyboard = GCKeyboard.coalesced?.keyboardInput {
        if keyboard.button(forKeyCode: .keyW)?.isPressed ?? false { moveUp() }
        if keyboard.button(forKeyCode: .keyA)?.isPressed ?? false { moveLeft() }
        if keyboard.button(forKeyCode: .keyS)?.isPressed ?? false { moveDown() }
        if keyboard.button(forKeyCode: .keyD)?.isPressed ?? false { moveRight() }
        runModifier = (keyboard.button(forKeyCode: .leftShift)?.value ?? 0) + 1.0
    }
}
```

Mouse delta handler and scroll wheel:
```swift
var mouseDelta = CGPoint.zero

if let mouse = GCMouse.current, let mouseInput = mouse.mouseInput {
    mouseInput.mouseMovedHandler = { (mouse, deltaX, deltaY) in
        self.mouseDelta = CGPoint(x: CGFloat(deltaX), y: CGFloat(deltaY))
    }
    mouseInput.leftButton.valueChangedHandler = { (_, _, pressed) in
        if pressed { self.attack() }
    }
    mouseInput.scroll.valueChangedHandler = { (_, x, y) in
        camera.fieldOfView = max(30, min(120, camera.fieldOfView + CGFloat(y)))
    }
}

// Apply delta in update loop, then clear
cameraDirection += simd_make_float2(-Float(mouseDelta.x * 0.02),
                                     Float(mouseDelta.y * 0.02))
mouseDelta = .zero
```

Lock the pointer to the app:
```swift
class GameViewController: UIViewController {
    private var pointerLocked = true

    override var prefersPointerLocked: Bool { pointerLocked }

    func toggleMouseMode() {
        pointerLocked.toggle()
        setNeedsUpdateOfPrefersPointerLocked()
    }
}
```

## Takeaways
- `GCKeyboard` and `GCMouse` are the right APIs for full-screen games that need raw delta events, custom pointer rendering, and suppression of system gestures; use UIKit pointer APIs for games that can tolerate standard system pointer behavior.
- Keyboard polling (`button(forKeyCode:)?.isPressed`) is O(1), non-blocking, and thread-safe — safe to call from render threads without stalling the pipeline.
- `mouseMovedHandler` delivers raw device deltas (not screen coordinates); the app is responsible for applying input acceleration and managing cursor position, giving complete control over feel.
- Set `prefersPointerLocked = true` in the game's root `UIViewController` to hide the system pointer and prevent Dock/Control Center/multitasking from triggering at screen edges during gameplay; toggle it to return to system UI.
- All connected keyboards are coalesced automatically into `GCKeyboard.coalesced`; for mice, use `GCMouseDidConnectNotification` to set up per-device handlers or just use `GCMouse.current` for single-mouse games.

---
_Source: WWDC20 Session 10617 page (abstract, transcript, and code samples)._
