# Make Your Game Great with Touch
**WWDC26 · Session 358** · [Watch](https://developer.apple.com/videos/play/wwdc2026/358/)

_Platforms:_ iOS, iPadOS

## Overview
This session provides a deep dive into building first-class touch controls for games on iPhone and iPad using the Touch Controller framework. Using the indie title Dredge by Black Salt Games as a recurring example alongside indie and AAA insights, the session covers the full design and implementation arc: setting up a touch controller, designing layouts that work across all screen sizes, making interactions feel fluid and native rather than like a controller overlay, and providing clear player feedback.

The Touch Controller framework integrates directly with the existing `GCController` ecosystem: a `TCTouchController` instance surfaces as a standard `GCController` to the rest of the game code, requiring minimal changes to existing input handling logic. The framework handles rendering the control overlays on top of the Metal view, hit testing, and delivering touch events as standard `GCControllerButtonInput` and thumbstick values.

The session emphasizes that great touch controls are not direct translations of physical controller layouts. The best experiences take advantage of the full screen, adjust contextually based on game state, hide controls that are irrelevant, and use the touchscreen's unique capabilities (relative touch deltas, large collider surfaces) to make the experience feel purposefully designed for touch.

## Key Topics

### Setting Up a Touch Controller (1:42)
A `TCTouchController` is created from a `TCTouchControllerDescriptor` that references the `MTKView`. After calling `connect()` it appears in `GCController.controllers()` like any physical controller. Rendering the overlay is done by calling `render(using:)` on the render encoder each frame. Touch events are forwarded from standard UIKit touch callbacks (`touchesBegan`, `touchesMoved`, etc.) to the touch controller's input handlers.

### Designing Flexible Layouts (4:52)
The framework provides nine anchor points for positioning controls. Developers read UIKit safe area insets and apply them to offsets to ensure controls stay within the safe zone on all device sizes and orientations. General placement principles: high-frequency actions (move, jump, attack) belong near the thumbs at screen edges; low-frequency actions (pause, map) belong at the top where they are not accidentally triggered. Controls can be grouped into sections that move together.

### Designing Fluid Interactions (10:17)
Key techniques for interaction quality:
- **Contextual icons**: Change the button B icon dynamically (e.g., to `figure.fencing`, `flame.fill`, `drop.fill`) as the active power changes, so the button always communicates its current action.
- **Hide unavailable controls**: Use `isEnabled = false` / `removeControl(_:)` to hide buttons that are not relevant in the current context. Show pickup buttons only when an item is nearby; hide power wheel buttons when not in combat.
- **Full-screen thumbstick colliders**: Set `colliderShape = .leftSide` on a thumbstick to make the entire left half of the screen the touch target, allowing players to place their thumb anywhere naturally.
- **Sprint via magnitude**: Calculate tilt magnitude (`simd_length(moveInput)`) to detect full-deflection sprinting intent without a separate sprint button.
- **Touchpad for camera**: Replace the right thumbstick with a `TCTouchpad` using `reportsRelativeValues = true` for smooth, cursor-like camera panning without a fixed anchor point.
- **Auto-hide thumbstick**: Set `hidesWhenNotPressed = true` on a thumbstick descriptor so the stick only appears when the player touches that area, reducing visual clutter.

### Providing Rich Feedback (21:16)
Built-in press states provide visual feedback automatically. Custom `TCControlContents` and `TCControlImage` can add layered visual effects—such as a glowing halo texture rendered behind the thumbstick background during sprint—by compositing custom images with framework-provided background images. Complex multi-input patterns (QTE requiring multiple simultaneous buttons, hold-and-aim-then-release) should be simplified into single contextual controls.

## APIs & Frameworks

### Touch Controller Framework (TCTouchController) — NEW
- **[NEW]** `TCTouchController` — main class representing the virtual touch gamepad
- **[NEW]** `TCTouchControllerDescriptor` — configuration object; initialized with `MTKView`
- **[NEW]** `TCTouchController.isSupported` — class property checking device support
- **[NEW]** `TCTouchController.connect()` — registers the controller with GCController system
- **[NEW]** `TCTouchController.render(using:)` — renders control overlay via `MTLRenderCommandEncoder`
- **[NEW]** `TCButtonDescriptor` — descriptor for configuring a virtual button
- **[NEW]** `TCThumbstickDescriptor` — descriptor for configuring a virtual thumbstick
- **[NEW]** `TCTouchpadDescriptor` — descriptor for configuring a virtual touchpad
- **[NEW]** `TCControlLabel` — identifies a control by role (e.g., `.buttonA`, `.buttonB`, `.buttonY`, `.rightThumbstick`)
- **[NEW]** `TCControlLabel(name:role:)` — creates a custom-named control label with a role
- **[NEW]** `TCControlLayoutAnchor` — nine anchor points for control placement (e.g., `.bottomRight`, `.bottomLeft`, `.topCenter`)
- **[NEW]** `TCControlContents` — encapsulates the visual contents of a control
- **[NEW]** `TCControlContents.buttonContents(forSystemImageNamed:size:shape:controller:)` — creates button contents from an SF Symbol
- **[NEW]** `TCControlContents.thumbstickStickBackgroundContents(size:controller:)` — standard thumbstick background contents
- **[NEW]** `TCControlContents(images:)` — creates custom contents from an array of `TCControlImage`
- **[NEW]** `TCControlImage` — a single image layer; properties: `texture`, `size`, `highlight`, `offset`, `tintColor`
- **[NEW]** `TCTouchController.addButton(descriptor:)` — adds a button control
- **[NEW]** `TCTouchController.addThumbstick(descriptor:)` — adds a thumbstick control
- **[NEW]** `TCTouchController.addTouchpad(descriptor:)` — adds a touchpad control
- **[NEW]** `TCTouchController.removeControl(_:)` — removes a control at runtime
- **[NEW]** `TCTouchController.buttons` — collection of current button controls
- **[NEW]** `TCThumbstickDescriptor.hidesWhenNotPressed` — Bool; hides thumbstick when no touch detected
- **[NEW]** `TCThumbstickDescriptor.colliderShape` — `.circle` or `.leftSide` / `.rightSide` for full-half-screen hit target
- **[NEW]** `TCTouchpadDescriptor.reportsRelativeValues` — Bool; reports delta rather than absolute position
- **[NEW]** `TCTouchpadDescriptor.colliderShape` — `.rightSide` for full right-half touch target
- **[NEW]** `GCControllerButtonInput.isEnabled` — Bool; enables or disables the button visually and functionally
- **[NEW]** `TCTouchController.backgroundContents` (on thumbstick) — custom `TCControlContents` for background layer

### Game Controller Framework (GCController)
- `GCController` — unified controller type; `TCTouchController` surfaces as a `GCController`
- `GCExtendedGamepad` — extended gamepad profile accessed via `controller.extendedGamepad`
- `GCControllerButtonInput` — button element; `isPressed`, `value`, `pressedChangedHandler`, `valueChangedHandler`
- `GCControllerDirectionPad` (thumbstick) — `xAxis.value`, `yAxis.value`
- `GCPhysicalInputElement` — base type for input elements in change handlers
- `GCPressedStateInput` — protocol for pressed-state change handler closure signature

### UIKit
- `UITouch` — touch event; `location(in:)`, `previousLocation(in:)`
- `UIView.touchesBegan(_:with:)`, `touchesMoved(_:with:)` — forwarded to touch controller input handling
- Safe area insets — used to compute safe control offsets across device sizes and orientations

### Metal / MetalKit
- `MTKView` — provided to `TCTouchControllerDescriptor` as the render target
- `MTLRenderCommandEncoder` — passed to `TCTouchController.render(using:)` each frame
- `MTLTexture` — used as `TCControlImage` texture for custom visual effects

### simd
- `simd_make_float2(_:_:)` — constructs a 2D float vector from thumbstick axis values
- `simd_length(_:)` — computes thumbstick tilt magnitude for sprint detection

## Code Highlights

Setting up a touch controller:
```swift
let descriptor = TCTouchControllerDescriptor(mtkView: mtkView)
if TCTouchController.isSupported {
    touchController = TCTouchController(descriptor: descriptor)
}
touchController?.connect()
// In render loop:
touchController?.render(using: renderEncoder)
```

Full-screen left-side thumbstick collider:
```swift
let leftStickDesc = TCThumbstickDescriptor()
leftStickDesc.colliderShape = .leftSide
leftStickDesc.hidesWhenNotPressed = true
touchController.addThumbstick(descriptor: leftStickDesc)
```

Touchpad for smooth camera control:
```swift
let touchpadDesc = TCTouchpadDescriptor()
touchpadDesc.label = TCControlLabel.rightThumbstick
touchpadDesc.colliderShape = .rightSide
touchpadDesc.reportsRelativeValues = true
touchController.addTouchpad(descriptor: touchpadDesc)
```

Custom glowing thumbstick background:
```swift
let haloLayer = TCControlImage(texture: haloTexture, size: haloSize,
                               highlight: nil, offset: .zero, tintColor: tint)
let normalBgImages = TCControlContents.thumbstickStickBackgroundContents(
    size: bgSize, controller: controller).images
haloThumbstickBg = TCControlContents(images: [haloLayer] + normalBgImages)
thumbstick.backgroundContents = active ? haloThumbstickBg : normalThumbstickBg
```

## Takeaways
- `TCTouchController` integrates with `GCController` so existing controller input code requires minimal changes; the framework handles overlay rendering and hit testing.
- Full-screen collider shapes (`.leftSide`, `.rightSide`) and relative-value touchpads are the most impactful changes for making touch feel native vs. awkward.
- Contextual controls—showing, hiding, and relabeling buttons based on game state—dramatically reduce screen clutter and player confusion without adding extra buttons.
- Custom `TCControlContents` with layered `TCControlImage` textures allow pixel-perfect branded visuals that match the game's art style while using the framework's built-in interaction model.

---
_Source: WWDC26 Session 358 page (abstract, chapter summaries, and code samples)._
