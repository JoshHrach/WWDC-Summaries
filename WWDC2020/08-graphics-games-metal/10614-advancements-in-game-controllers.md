# Advancements in Game Controllers
**WWDC20 · Session 10614** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10614/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14

## Overview
This session covers significant expansions to the Game Controller framework in iOS 14 and related platforms, including new controller support, an extensible input API, Core Haptics integration for rumble feedback, and a new system-level input remapping feature. Two newly supported controllers are announced: the Xbox Elite Wireless Controller Series 2 and the Xbox Adaptive Controller, which helps make gaming more accessible.

The Extensible Input API (`GCPhysicalInputProfile`) allows games to dynamically query all inputs on any connected controller at runtime — including non-standard inputs like the DualShock 4 touchpad and Xbox Elite paddle buttons — without needing controller-specific code paths. Core Haptics integration with `GCController` brings programmable rumble to supported controllers via AHAP patterns, enabling consistent haptic design across iOS device haptics and controller motors.

A new system-level input remapping feature (accessible via iOS Settings > General > Game Controller and tvOS Settings > Remotes and Devices > Bluetooth) lets users remap buttons globally or per-app without requiring any in-app implementation, as long as the app declares game controller support via Xcode capabilities.

## Key Topics
- **New controllers** — Xbox Elite Wireless Controller Series 2 and Xbox Adaptive Controller now supported.
- **`GCPhysicalInputProfile`** **[NEW]** — Base class for all controller input collections; every `GCController` now exposes a `physicalInputProfile`; enables runtime discovery of all inputs including paddles, touchpads.
- **`GCDualShockGamepad`** **[NEW]** — `GCExtendedGamepad` subclass exposing the DualShock 4 touchpad (two-finger tracking + button).
- **`GCXboxGamepad`** **[NEW]** — `GCExtendedGamepad` subclass exposing the four paddle buttons of the Xbox Elite Wireless Controller.
- **Core Haptics + Game Controller integration** **[NEW]** — `GCController.haptics?.createEngine(withLocality:)` returns a `CHHapticEngine` targeting specific physical locations on a controller (handles, left trigger, right trigger); play AHAP patterns on controllers.
- **`GCHapticsLocality`** **[NEW]** — Enum specifying which physical motor(s) to target: `.default`, `.handles`, `.leftHandle`, `.rightHandle`, `.leftTrigger`, `.rightTrigger`.
- **Dynamic haptic updates** — Use `CHHapticDynamicParameter` with `hapticIntensityControl` to update motor intensity every frame without creating new pattern players.
- **`GCController.current`** **[NEW]** — Class property returning the most recently active controller; intended for single-player games.
- **`GCControllerDidBecomeCurrent` / `GCControllerDidStopBeingCurrent`** **[NEW]** — Notifications for tracking the active controller.
- **`GCMotion` enhancements** — `sensorsRequireManualActivation`, `sensorsActive`, `hasGravityAndUserAcceleration` for DualShock 4 gyroscope/accelerometer support.
- **`GCDeviceLight`** **[NEW]** — `GCController.light` property for controlling the DualShock 4 lightbar color via `GCColor`.
- **`GCDeviceBattery`** **[NEW]** — `GCController.battery` property; `batteryLevel: Float`, `batteryState: GCDeviceBattery.State`; supports KVO.
- **System input remapping** — Available in iOS 14 and tvOS 14; enabled by declaring game controller support in Xcode capabilities; global and per-app remapping.
- **Controller button glyphs in SF Symbols** **[NEW]** — `GCControllerElement.sfSymbolsName` returns the correct SF Symbol name for the physical button (e.g., Y-button vs. triangle for DualShock 4); automatically reflects user input remapping.

## APIs & Frameworks

### Game Controller (GameController.framework)
- **`GCPhysicalInputProfile`** **[NEW]** — `allButtons: Set<GCControllerButtonInput>`, `buttons: [String: GCControllerButtonInput]`, subscript by element name string
- **`GCDualShockGamepad`** **[NEW]** — Subclass of `GCExtendedGamepad`; `touchpadButton`, `touchpadPrimary`, `touchpadSecondary`
- **`GCXboxGamepad`** **[NEW]** — Subclass of `GCExtendedGamepad`; `paddleButton1`, `paddleButton2`, `paddleButton3`, `paddleButton4`
- **`GCInputDualShockTouchpadButton`** **[NEW]** — String constant for keying into `physicalInputProfile.buttons`
- **`GCController.current`** **[NEW]** — `class var current: GCController?`
- **`GCController.haptics`** **[NEW]** — `GCControllerHaptics?`; `.createEngine(withLocality:) -> CHHapticEngine`
- **`GCHapticsLocality`** **[NEW]** — `.default`, `.handles`, `.leftHandle`, `.rightHandle`, `.leftTrigger`, `.rightTrigger`, `.all`
- **`GCDeviceLight`** **[NEW]** — `GCController.light`; `color: GCColor`
- **`GCColor`** **[NEW]** — `init(red:green:blue:)`
- **`GCDeviceBattery`** **[NEW]** — `GCController.battery`; `batteryLevel: Float`, `batteryState: GCDeviceBattery.State`
- **`GCDeviceBattery.State`** **[NEW]** — `.discharging`, `.charging`, `.full`, `.unknown`
- **`GCControllerElement.sfSymbolsName`** **[NEW]** — `String?`; SF Symbol name for the element, respecting remapping
- **`GCMotion.sensorsRequireManualActivation`** — `Bool`
- **`GCMotion.sensorsActive`** — `Bool` (settable)
- **`GCMotion.hasGravityAndUserAcceleration`** — `Bool`

### Core Haptics (CoreHaptics.framework)
- **`CHHapticEngine`** — `makePlayer(with:) -> CHHapticPatternPlayer`; `start(completionHandler:)`
- **`CHHapticPattern`** — `init(events:parameters:)`
- **`CHHapticEvent`** — `init(eventType:parameters:relativeTime:duration:)` (continuous); `init(eventType:parameters:relativeTime:)` (transient)
- **`CHHapticEvent.EventType`** — `.hapticContinuous`, `.hapticTransient`
- **`CHHapticEventParameter`** — `init(parameterID:value:)`; `.hapticSharpness`, `.hapticIntensity`
- **`CHHapticPatternPlayer`** — `start(atTime:)` with `CHHapticTimeImmediate`; `sendParameters(_:atTime:)`
- **`CHHapticDynamicParameter`** — `init(parameterID:value:relativeTime:)`; `.hapticIntensityControl` for per-frame motor updates

## Code Highlights

Accessing non-standard inputs via extensible input API:
```swift
let input = controller.physicalInputProfile
let paddleBtn = input.buttons["Paddle 1"]
let touchpadBtn = input.buttons[GCInputDualShockTouchpadButton]
let unmapped = input.allButtons.filter { !mappedButtons.contains($0) }
```

Creating a haptic engine targeting controller handles:
```swift
hapticEngine = controller.haptics?.createEngine(withLocality: .handles)
```

Per-frame dynamic haptic intensity update:
```swift
let intensityParam = CHHapticDynamicParameter(
    parameterID: .hapticIntensityControl, value: motorIntensity, relativeTime: 0)
try patternPlayer?.sendParameters([intensityParam], atTime: CHHapticTimeImmediate)
```

Reading the correct SF Symbol glyph for a button (respects remapping):
```swift
let sfName = controller.physicalInputProfile[GCInputButtonY]?.sfSymbolsName
let glyph = sfName.map { UIImage(systemName: $0) }
```

## Takeaways
- The new Extensible Input API (`GCPhysicalInputProfile`) enables runtime discovery of all controller inputs including non-standard ones; `GCDualShockGamepad` and `GCXboxGamepad` provide typed access to platform-specific inputs.
- Core Haptics integration allows designing AHAP haptic patterns once and playing them on any supported controller motor; use `GCHapticsLocality` to target specific handles or triggers.
- Enable the Game Controllers capability in Xcode to get App Store badging, system-level input remapping (iOS 14 / tvOS 14), and per-app remapping when "Extended Gamepad" is also checked.
- Use `GCControllerElement.sfSymbolsName` to display the correct controller button glyph in UI — it automatically reflects the user's input remapping configuration.

---
_Source: WWDC20 Session 10614 page (abstract, chapter summaries, code samples, and resource links)._
