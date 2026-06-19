# Bring Your iOS or iPadOS Game to visionOS
**WWDC24 · Session 10093** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10093/)

_Platforms:_ visionOS 1.x / visionOS 2, iOS, iPadOS (porting context)

## Overview
This session explains how to bring an existing iOS or iPadOS game to visionOS, with a focus on the Compatible Mode approach (running your iOS app in a window on visionOS) versus building a native visionOS experience. The session walks through what "just works," what needs attention, and what can be enhanced to take advantage of the visionOS environment.

The compatible mode path lets iOS games run on visionOS with minimal changes, but the session shows how to progressively enhance them: adopting the Game Controller framework for spatial input, adding support for the visionOS hover effect, integrating with RealityKit for mixed-reality overlays, and considering the unique visual and interaction design affordances of spatial computing. The session uses a real game (Blackbird) as its demonstration vehicle.

## Key Topics
- **Compatible Mode** — iOS apps run on visionOS as a flat window in the shared space; most rendering APIs work as-is
- **Input** — `GCController` and `GCVirtualController` work on visionOS; spatial tap via `UIResponder` touch events mapped to gaze+pinch; Game Controller framework is the recommended input path
- **Rendering compatibility** — Metal, SpriteKit, SceneKit, and UIKit render correctly in compatible mode; verify frame rate and resolution targets
- **Hover effects** — `UIHoverGestureRecognizer` available on visionOS for gaze-hover feedback
- **Game Controller enhancements** — `GCController.current` on visionOS maps to gaze+pinch or connected gamepad; `GCVirtualController` provides on-screen controls in compatible mode
- **Progressive enhancement** — detect visionOS via `UIDevice.current.userInterfaceIdiom == .vision` or `#if os(visionOS)`; conditionally add spatial features
- **RealityKit integration** — add a `RealityView` alongside a UIKit game view to add 3D/AR overlays
- **Fully native path** — SwiftUI + RealityKit + immersive spaces for games that want to go fully spatial; recommended for new development targeting visionOS exclusively

## APIs & Frameworks
### Game Controller
- `GCController` — enumerate connected controllers; on visionOS, maps to gaze+pinch as a virtual input
- `GCVirtualController` — on-screen controller overlay; works on visionOS compatible mode
- `GCController.controllerDidConnectNotification` — respond to controller connect/disconnect
- `GCExtendedGamepad` — thumbstick, buttons, triggers; full gamepad profile
- **[NEW visionOS 2] `GCController.input` spatial mapping** — improved gaze-as-pointer semantics for game menus

### UIKit (compatible mode)
- `UIHoverGestureRecognizer` — fires on visionOS gaze hover; use for highlight effects on interactive elements
- `UITouch.type == .indirectPointer` — identify visionOS indirect tap input
- `traitCollection.userInterfaceIdiom == .vision` — detect visionOS at runtime

### Metal / Rendering
- `MTKView` / `CAMetalLayer` — work in compatible mode; target 90 fps for visual comfort
- `MTLDevice` — Apple Silicon GPU with unified memory on visionOS; optimize for TBDR architecture
- `MTLRenderPassDescriptor` — standard Metal rendering pipeline; no changes needed for basic compatibility

### RealityKit (progressive enhancement)
- `RealityView` — embed 3D RealityKit content alongside UIKit game views
- `AnchorEntity(.plane(.horizontal))` — place 3D content on surfaces detected by visionOS
- `ImmersiveSpace` — for fully spatial game experiences; requires SwiftUI app structure

### visionOS Specific
- `#if os(visionOS)` — compile-time conditional for platform-specific code
- `UIApplication.shared.openURL` — behavior differences on visionOS; test all deep links
- `AVAudioSession` — spatial audio routing automatically enhanced on visionOS; no API changes needed

## Code Highlights
```swift
// Detect visionOS at runtime for progressive enhancement
#if os(visionOS)
func addHoverEffect(to button: UIButton) {
    let hover = UIHoverGestureRecognizer(target: self, action: #selector(handleHover))
    button.addGestureRecognizer(hover)
}
#endif

// Game Controller input on visionOS
let controller = GCController.controllers().first
controller?.extendedGamepad?.buttonA.valueChangedHandler = { button, value, pressed in
    if pressed { handleJump() }
}

// Virtual controller for compatible mode touch games
let config = GCVirtualController.Configuration()
config.elements = [GCInputButtonA, GCInputButtonB, GCInputLeftThumbstick]
let virtualController = GCVirtualController(configuration: config)
virtualController.connect()
```

## Takeaways
- The compatible mode path (submitting an iOS app targeting visionOS) is viable for most 2D and UIKit-based games with minimal code changes; Metal and SpriteKit rendering works as-is
- Adopt `UIHoverGestureRecognizer` and `GCController` as the first progressive enhancements—they dramatically improve the visionOS feel with small code changes
- Target 90 fps; the visionOS display's 90Hz panel makes frame rate drops more perceptible than on flat screens
- For games that want to go fully spatial, plan a native visionOS target with SwiftUI + RealityKit + Immersive Space rather than continuing to layer onto the compatible mode path

---
_Source: WWDC24 Session 10093 page (abstract, chapter summaries, code samples, and resource links)._
