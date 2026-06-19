# Supporting New Game Controllers
**WWDC19 · Session 616** · [Watch](https://developer.apple.com/videos/play/wwdc2019/616/)

_Platforms:_ iOS 13, macOS Catalina 10.15, tvOS 13

## Overview
iOS 13, macOS Catalina, and tvOS 13 expand the Game Controller framework to support two major new console controllers: the Microsoft Xbox Wireless controller and the Sony DualShock 4. Games already using the Game Controller framework automatically gain support for these controllers with no code changes required.

This session covers how to access new controller inputs (menu/options buttons, clickable thumbsticks), how to adapt UI assets to match the active controller type, and why macOS games using lower-level APIs like IOKit should migrate to the Game Controller framework.

## Key Topics

### Newly Supported Controllers **[NEW]**
- **Xbox Wireless Controller** (Bluetooth) — automatically supported in any Game Controller framework game.
- **DualShock 4** — automatically supported in any Game Controller framework game.
- Both use the existing `GCExtendedGamepad` profile — no new profiles needed.
- All three controller types (MFi, Xbox Wireless, DualShock 4) are instances of `GCController`.

### Connecting and Detecting Controllers
- `GCController.controllers()` — returns currently connected controllers at launch.
- Observe `GCControllerDidConnectNotification` and `GCControllerDidDisconnectNotification` for runtime connect/disconnect events.
- Register observers in `application(_:didFinishLaunchingWithOptions:)`.

### Button Mapping via Positional Equivalents
- DualShock 4 uses symbols (cross, circle, triangle, square) rather than letters — mapped positionally to `GCExtendedGamepad` properties:
  - Cross → `buttonA`, Circle → `buttonB`, Square → `buttonX`, Triangle → `buttonY`
- Xbox Wireless button labels (A/B/X/Y) map directly.

### New Auxiliary Buttons **[NEW]**
- `GCExtendedGamepad.buttonMenu` — the right auxiliary (pause) button; present on all supported controllers. **[NEW]**
  - Xbox: Menu button; DualShock 4: Options button; MFi: center/menu button.
  - Use this instead of the now-deprecated `GCController.controllerPausedHandler`.
- `GCExtendedGamepad.buttonOptions` — the left auxiliary button; NOT present on all controllers. **[NEW]**
  - Xbox: View button; DualShock 4: Share button.
  - Always check if `buttonOptions != nil` before using; provide alternative access via the pause menu if absent.
- Xbox button and PS button are reserved for system use.
- `GCController.controllerPausedHandler` — deprecated; migrate to `buttonMenu`. **[DEPRECATED]**

### Clickable Thumbsticks (L3/R3)
- Supported since iOS 12.1 / tvOS 12.1 / macOS 10.14.1.
- `GCExtendedGamepad.leftThumbstickButton` / `rightThumbstickButton` — may be `nil` on some MFi controllers.
- Always check for `nil` before assigning a handler; provide an alternative input path if absent.

### `GCController.productCategory` **[NEW]**
- New property that identifies the controller type: MFi, Xbox, or DualShock 4.
- Use to select appropriate button artwork or display names in UI prompts.
- Fall back to generic/MFi assets for unknown future controller types.

### UI Best Practices for Multiple Controller Types
- Button glyphs differ between controller families — adapt in-game prompts to match the active controller.
- Alternative: use position-based generic symbols ("right face button") instead of controller-specific glyphs.
- When multiple controllers are connected, track input from all of them and update UI to match the most-recently-used controller.

### macOS Migration from IOKit
- Games previously using IOKit for controller support should migrate to the Game Controller framework.
- Using both simultaneously creates conflicts (controllers appear in both systems).
- The Game Controller framework provides a consistent API across iOS, tvOS, and macOS and gains new controller support automatically.

## APIs & Frameworks

### Game Controller Framework **[UPDATED]**
- `GCController` — represents any connected controller (MFi, Xbox Wireless, DualShock 4)
- `GCController.controllers() -> [GCController]` — currently connected controllers
- `GCControllerDidConnectNotification` — controller connected notification
- `GCControllerDidDisconnectNotification` — controller disconnected notification
- `GCController.productCategory: String` — controller family identifier **[NEW]**
- `GCExtendedGamepad` — standard extended gamepad profile
  - `buttonA`, `buttonB`, `buttonX`, `buttonY` — face buttons
  - `leftThumbstick`, `rightThumbstick` — analog sticks
  - `leftThumbstickButton`, `rightThumbstickButton` — clickable thumbstick buttons (may be nil)
  - `leftShoulder`, `rightShoulder` — shoulder buttons
  - `leftTrigger`, `rightTrigger` — analog triggers
  - `dpad` — directional pad
  - `buttonMenu: GCControllerButtonInput` — pause/menu button **[NEW]**
  - `buttonOptions: GCControllerButtonInput?` — options/share/view button (optional) **[NEW]**
- `GCControllerButtonInput.pressedChangedHandler` — press/release callback
- `GCControllerButtonInput.valueChangedHandler` — analog value callback
- `GCController.controllerPausedHandler` — **deprecated**, use `buttonMenu` instead **[DEPRECATED]**

## Code Highlights

Detecting and observing controllers:
```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    NotificationCenter.default.addObserver(self, selector: #selector(controllerConnected), name: .GCControllerDidConnect, object: nil)
    NotificationCenter.default.addObserver(self, selector: #selector(controllerDisconnected), name: .GCControllerDidDisconnect, object: nil)
    for controller in GCController.controllers() {
        setupController(controller)
    }
    return true
}
```

Handling the menu button (with fallback):
```swift
if #available(iOS 13, *) {
    controller.extendedGamepad?.buttonMenu.pressedChangedHandler = { _, _, pressed in
        if pressed { togglePauseMenu() }
    }
} else {
    controller.controllerPausedHandler = { _ in togglePauseMenu() }
}
```

Checking for optional buttons:
```swift
if let optionsButton = controller.extendedGamepad?.buttonOptions {
    optionsButton.pressedChangedHandler = { _, _, pressed in
        if pressed { showSettingsMenu() }
    }
} else {
    // Ensure settings are accessible via pause menu
}
```

Selecting button artwork by controller type:
```swift
func getBlockButtonAsset(for controller: GCController) -> UIImage {
    switch controller.productCategory {
    case GCProductCategoryXboxOne:
        return UIImage(named: "xbox_b_red")!
    case GCProductCategoryDualShock4:
        return UIImage(named: "ps4_circle_red")!
    default:
        return UIImage(named: "mfi_b_green")!
    }
}
```

## Takeaways
- Xbox Wireless and DualShock 4 support is free for any app already using the Game Controller framework — no code changes needed.
- Migrate `controllerPausedHandler` to `GCExtendedGamepad.buttonMenu` (new in iOS 13); always guard `buttonOptions` with a nil check since not all controllers have it.
- Use `GCController.productCategory` to select controller-appropriate button glyphs in UI prompts; fall back to generic positional symbols for future-proofing.
- macOS games using IOKit for controller input should migrate fully to the Game Controller framework to avoid conflicts and gain automatic future controller support.

---
_Source: WWDC19 Session 616 page (abstract, chapter summaries, code samples, and resource links)._
