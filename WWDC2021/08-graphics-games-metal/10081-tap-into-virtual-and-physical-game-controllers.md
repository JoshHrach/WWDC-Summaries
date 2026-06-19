# Tap into Virtual and Physical Game Controllers
**WWDC21 · Session 10081** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10081/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
This session covers improvements to game input across Apple platforms, with three main areas: a new virtual on-screen controller system, support for next-generation physical controllers including the Sony DualSense with adaptive triggers, and new media capture (highlight clip) functionality via controller Share buttons.

The Game Controller framework's goal is providing a single, consistent API across all input types — physical controllers, keyboards, and mice — so developers write input handling code once. The new virtual on-screen controller (`GCVirtualController`) is a key addition that converts touch input into standard `GCController` events, allowing games designed for physical controllers to work naturally on iPhone and iPad without separate touch input code paths.

Physical controller support now includes the Sony DualSense and Xbox Series X controllers, introduced in iOS 14.5/macOS 11.3. DualSense adaptive triggers can simulate physical sensations like tension and vibration, enhancing immersion. The session also covers App Store badging, per-app controller customization, and best practices for displaying controller glyphs.

## Key Topics

### Game Controller Framework Basics
- `GCController`, `GCKeyboard`, `GCMouse` as `GCDevice` subtypes
- Connect/disconnect notifications: `GCControllerDidConnect`, `GCControllerDidDisconnect`
- Polling vs. value-changed handlers for input state
- `physicalInputProfile` for accessing button/axis state
- Adding the Game Controllers capability in Xcode for App Store badging and per-app customization

### Virtual On-Screen Controller (GCVirtualController)
- `GCVirtualController` **[NEW]**: renders touch controls that appear as a standard `GCController` to game logic
- `GCVirtualControllerConfiguration` **[NEW]**: specifies which elements to include (thumbsticks, buttons, d-pad, touchpad)
- Left and right regions configured independently; layout determined by configuration (no manual placement)
- Up to 4 buttons per side plus one of thumbstick/d-pad/touchpad
- `GCVirtualController.connect()` triggers standard `GCControllerDidConnect` notification
- `GCVirtualController.changeElement(_:block:)` **[NEW]**: customize button appearance with `UIBezierPath`
- `GCVirtualControllerElementConfiguration.path` **[NEW]**: custom shape for on-screen button
- `GCVirtualControllerElementConfiguration.hidden` **[NEW]**: show/hide individual elements
- Haptic feedback built in for buttons and sticks
- Elements support: `GCInputLeftThumbstick`, `GCInputRightThumbstick`, `GCInputButtonA`, `GCInputButtonB`, `GCInputButtonX`, `GCInputButtonY`, `GCInputDirectionPad`, `GCInputTouchpadButton`

### Physical Controller Support
- Sony DualSense controller support **[NEW]** (iOS 14.5, macOS 11.3)
- Xbox Series X controller support **[NEW]** (iOS 14.5, macOS 11.3)
- Xbox Series X dedicated Share button
- `GCDualSenseGamepad` **[NEW]**: profile for DualSense controller
- `GCDualSenseAdaptiveTrigger` **[NEW]**: DualSense's left/right adaptive triggers
- `GCDualSenseAdaptiveTrigger.setModeFeedbackWithStartPosition(_:resistiveStrength:)` **[NEW]**: constant resistance effect
- `GCDualSenseAdaptiveTrigger.setModeVibrationWithStartPosition(_:amplitude:frequency:)` **[NEW]**: vibration effect
- `GCDualSenseAdaptiveTrigger.setModeOff()` **[NEW]**: disable adaptive trigger effect
- `GCDualSenseAdaptiveTrigger.mode` **[NEW]**: current mode (`.off`, feedback, vibration)
- `GCDualSenseAdaptiveTrigger.value` **[NEW]**: current trigger pull value (0.0–1.0)

### Controller Glyphs and UI
- `GCControllerElement.sfSymbolsName`: SF Symbol name for a button (respects system remapping)
- `UIImage(systemName:)`: display the correct glyph even when buttons are remapped
- Positional button location display as alternative to named glyphs

### Media Capture (Share Button)
- Double-press Share button: screenshot to Camera Roll/Desktop
- Long-press Share button: start/stop ReplayKit recording OR save 15-second highlight clip **[NEW]**
- Background buffering for 15-second highlight clips **[NEW]** — toggled in Game Controller preferences
- API to trigger highlight saves programmatically (see "Discover rolling clips with ReplayKit")

## APIs & Frameworks

- `GameController` framework
- `GCController`
- `GCController.current`
- `GCControllerDidConnect` (notification)
- `GCControllerDidDisconnect` (notification)
- `GCKeyboard`
- `GCMouse`
- `GCDevice`
- `GCPhysicalInputProfile` / `physicalInputProfile`
- `GCControllerElement.valueChangedHandler`
- `GCInputButtonA`, `GCInputButtonB`, `GCInputButtonX`, `GCInputButtonY`
- `GCInputLeftThumbstick`, `GCInputRightThumbstick`
- `GCInputDirectionPad`
- `GCControllerElement.sfSymbolsName`
- `GCVirtualController` **[NEW]**
- `GCVirtualControllerConfiguration` **[NEW]**
- `GCVirtualControllerElementConfiguration` **[NEW]**
- `GCVirtualControllerElementConfiguration.path` **[NEW]**
- `GCVirtualControllerElementConfiguration.hidden` **[NEW]**
- `GCDualSenseGamepad` **[NEW]**
- `GCDualSenseAdaptiveTrigger` **[NEW]**
- `GCDualSenseAdaptiveTrigger.setModeFeedbackWithStartPosition(_:resistiveStrength:)` **[NEW]**
- `GCDualSenseAdaptiveTrigger.setModeVibrationWithStartPosition(_:amplitude:frequency:)` **[NEW]**
- `GCDualSenseAdaptiveTrigger.setModeOff()` **[NEW]**
- `GCDualSenseAdaptiveTrigger.mode` **[NEW]**
- `GCDualSenseAdaptiveTrigger.value` **[NEW]**
- `GCDualShock4Gamepad`
- `GCXboxGamepad`

## Code Highlights

Creating a virtual on-screen controller:
```swift
let virtualConfiguration = GCVirtualControllerConfiguration()
virtualConfiguration.elements = [GCInputLeftThumbstick, GCInputRightThumbstick,
                                   GCInputButtonA, GCInputButtonB]
let virtualController = GCVirtualController(configuration: virtualConfiguration)
virtualController.connect()
```

DualSense adaptive trigger feedback for a slingshot mechanic:
```swift
guard let dualSense = GCController.current?.physicalInputProfile as? GCDualSenseGamepad else { return }
let adaptiveTrigger = dualSense.rightTrigger
if playerIsPullingSlingshot {
    let resistiveStrength = min(1, 0.4 + adaptiveTrigger.value)
    if adaptiveTrigger.value < 0.9 {
        adaptiveTrigger.setModeFeedbackWithStartPosition(0, resistiveStrength: resistiveStrength)
    } else {
        adaptiveTrigger.setModeVibrationWithStartPosition(0, amplitude: resistiveStrength, frequency: 0.03)
    }
} else if adaptiveTrigger.mode != .off {
    adaptiveTrigger.setModeOff()
}
```

## Takeaways

- Use `GCVirtualController` to give touch-based players a great on-screen controller experience that integrates seamlessly with existing `GCController` input code.
- Add the Game Controllers capability in Xcode to enable App Store badging, per-app input remapping, and automatic screenshot/clip capture via the Share button.
- Always query `GCControllerElement.sfSymbolsName` for glyphs so your UI reflects the player's actual (possibly remapped) controller buttons.
- Use `GCDualSenseAdaptiveTrigger` effects to add tactile immersion for DualSense users, remembering to call `setModeOff()` when the effect is no longer needed.

---
_Source: WWDC21 Session 10081 page (abstract, chapter summaries, code samples, and resource links)._
