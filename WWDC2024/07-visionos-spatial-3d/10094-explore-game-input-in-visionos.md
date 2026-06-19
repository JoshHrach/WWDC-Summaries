# Explore Game Input in visionOS
**WWDC24 · Session 10094** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10094/)

_Platforms:_ visionOS, visionOS 2

## Overview
visionOS offers three tiers of input for games: **system gestures** (the recommended default), **spatial events** (for precise 3D hit-testing and custom gesture recognition), and **game controllers** (via Game Controller framework). This session walks through each tier, explains their design tradeoffs, and shows how to combine them for a layered input strategy that feels natural to Vision Pro users while supporting external hardware.

The session also covers ARKit custom hand-gesture recognition using the full 26-joint skeleton — a lower-level path for games that need more expressive input than touch gestures can provide.

## Key Topics

### System Gestures (Recommended Baseline)
Standard SwiftUI gestures (`TapGesture`, `DragGesture`, `MagnifyGesture`, `RotateGesture3D`) work on RealityKit entities with `InputTargetComponent` + `CollisionComponent` attached. They handle eye + pinch automatically and compose with `.simultaneously(with:)` for multi-gesture states. No extra entitlements needed.

### SpatialEventGesture
`SpatialEventGesture` is a low-level SwiftUI gesture that fires raw pinch events with 3D location data. It provides `SpatialEventCollection` containing `SpatialEvent` values with:
- `id` — unique per pinch
- `location3D` — world-space position of the pinch
- `phase` — `.active`, `.ended`, `.cancelled`
- `modifierKeys` — connected keyboard modifier state

Use `SpatialEventGesture` for games needing precise world-space coordinates per pinch event, rather than per-entity hit testing.

### ARKit Full Hand Skeleton
For fully custom hand gestures, use ARKit `HandAnchor` with `HandSkeleton` (26 joints per hand). Common pattern: sample joint positions each frame, compute relative angles/distances, and match against a state machine. Requires the Hand Tracking entitlement and user permission. More expressive than pinch gestures — enables peace signs, thumbs-up, pointing, etc.

### Game Controller (GCController)
Standard `GCController` support works on visionOS: register for `GCControllerDidConnect` notification, query `controller.extendedGamepad`. All MFi and Made for Apple Silicon controllers work. Attach the `.handlesGameControllerEvents(matching: .gamepad)` view modifier to the scene's root view so the system routes controller input to the game instead of the OS.

### Input Design Principles
- Avoid requiring simultaneous two-handed pinches as primary input — ergonomically fatiguing.
- Prefer indirect input (eye + pinch at a distance) for menu navigation; reserve direct touch for close-range manipulation.
- Always provide a game controller path as an accessibility alternative.
- Test with eye tracking calibration to account for per-user gaze offsets.

## APIs & Frameworks

**SwiftUI / RealityKit gestures**
- `TapGesture`, `DragGesture`, `MagnifyGesture`, `RotateGesture3D` (existing, work with entities)
- `.simultaneously(with:)` gesture combinator (existing)
- `InputTargetComponent` — required for entity gesture targeting (existing)
- `CollisionComponent` — required for entity gesture targeting (existing)
- `SpatialEventGesture` **[NEW]**
  - `.onChanged { events in ... }` — `events: SpatialEventCollection`
- `SpatialEventCollection` **[NEW]**
- `SpatialEvent` **[NEW]**
  - `.id: SpatialEvent.ID`
  - `.location3D: Point3D`
  - `.phase: SpatialEvent.Phase` (`.active`, `.ended`, `.cancelled`)
  - `.modifierKeys: EventModifiers`

**ARKit**
- `HandAnchor` (existing) — position/orientation of the hand in world space
- `HandSkeleton` (existing) — 26 joints per hand
- `HandSkeleton.JointName` (existing) — named joint constants
- `HandTrackingProvider` (existing) — ARKit data source for hand anchors

**Game Controller**
- `GCController` (existing)
- `GCControllerDidConnect` notification (existing)
- `GCExtendedGamepad` (existing)

**SwiftUI (visionOS modifier)**
- `.handlesGameControllerEvents(matching:)` modifier **[NEW]**
  - Parameter: `GCControllerElement.Category` (e.g., `.gamepad`)

## Code Highlights

```swift
// System gesture on a RealityKit entity
entity.components.set(InputTargetComponent())
entity.components.set(CollisionComponent(shapes: [.generateBox(size: .one)]))

RealityView { content in
    content.add(entity)
}
.gesture(TapGesture().targetedToEntity(entity).onEnded { _ in
    entity.components[PhysicsBodyComponent.self]?.applyImpulse(.up)
})

// SpatialEventGesture for world-space pinch tracking
.gesture(SpatialEventGesture()
    .onChanged { events in
        for event in events where event.phase == .active {
            handlePinch(at: event.location3D)
        }
    }
)

// Game controller
NotificationCenter.default.addObserver(forName: .GCControllerDidConnect, object: nil, queue: .main) { note in
    let controller = note.object as! GCController
    controller.extendedGamepad?.buttonA.pressedChangedHandler = { _, _, pressed in
        if pressed { jump() }
    }
}

// Route controller input to the game
ContentView()
    .handlesGameControllerEvents(matching: .gamepad)
```

## Takeaways
- Start with system `SwiftUI` gestures + `InputTargetComponent` before reaching for lower-level APIs — they handle eye tracking, hit testing, and composability automatically.
- Use `SpatialEventGesture` when you need raw world-space pinch coordinates rather than per-entity events, such as for drawing or trajectory-based mechanics.
- Implement `GCController` support even if you target gaze+pinch primarily — it is an important accessibility path and enables Apple TV/Mac crossover.
- Add `.handlesGameControllerEvents(matching: .gamepad)` at the root of your scene so controller buttons are not intercepted by system shortcuts.

---
_Source: WWDC24 Session 10094 page (abstract, chapter summaries, code samples, and resource links)._
