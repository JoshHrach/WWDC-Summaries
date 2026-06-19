# Design Interactive Experiences for visionOS
**WWDC24 · Session 10096** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10096/)

_Platforms:_ visionOS 2

## Overview
This session is a design deep dive into making interactions in visionOS apps feel natural, intentional, and satisfying. Building on the Human Interface Guidelines for visionOS, the talk focuses on three interaction modes — gaze + pinch (indirect), direct touch, and gestures in immersive spaces — and explains what makes each feel right vs. broken. The session is organized around case studies drawn from first-party and partner apps, annotating both good patterns and common pitfalls with before/after comparisons.

Key themes include: calibrating target sizes for reliable eye-tracking activation, the role of audio feedback in confirming interactions, and how to design gestures in fully immersive spaces where hand tracking is the only input. The session also introduces updated guidance for the new visionOS 2 hover effect system.

## Key Topics
- **Gaze + pinch interaction model** — the user looks at a target to aim, then pinches to activate; targets must be at least 60 pt and have a clear hover state; avoid overlapping targets.
- **Direct touch in visionOS** — reaching out and touching virtual surfaces with fingers; effective for close-range manipulation of 3D objects and volumes; pair with `accessibilityDirectTouch` for VoiceOver.
- **Gesture design for immersive spaces** — with no visible UI chrome, gestures must be discoverable through onboarding; recommend no more than 2–3 custom gestures per immersive experience; always provide a fallback (pinch or tap).
- **Audio feedback** — system sounds (clicks, swooshes) confirm interactions; add custom `UIFeedbackGenerator` / `AVAudioPlayer` cues for complex multi-step interactions.
- **Custom hover effects (visionOS 2)** — the new `HoverEffect` closure API (see session 10152) should be used consistently; every interactive element must have a visible hover state.
- **Comfort and fatigue** — avoid interactions that require sustained arm lifting (gorilla arm); favor wrist-level pinch gestures and gaze-aim over fully extended arm pointing.

## APIs & Frameworks

**SwiftUI / visionOS**
- `hoverEffect(_:)` — apply system highlight to any interactive view; required for all tappable UI
- **[NEW]** `HoverEffect` closure — custom hover effect via `hoverEffect { proxy, isActive, _ in … }` (see session 10152)
- `onTapGesture(count:perform:)` — gaze + pinch tap; single or double
- `simultaneousGesture(_:)` — layer gestures without blocking; use with `DragGesture` in volumes
- `SpatialTapGesture` — tap with 3D location info; `value.location3D` gives position in scene space
- `DragGesture(minimumDistance:coordinateSpace:)` — drag in 2D or 3D; use `.local` coordinate space in volumes
- `MagnifyGesture` / `RotateGesture3D` — pinch-to-scale and rotation in 3D
- `accessibilityDirectTouch(isEnabled:options:)` — pass raw touch to VoiceOver for direct-touch surfaces
- `PhysicalMetricsConverter` — convert logical points to physical centimeters for minimum target size calculations
- `UIImpactFeedbackGenerator` / `UISelectionFeedbackGenerator` — haptic feedback on supported controllers; not on Vision Pro hardware directly but important for paired device flows
- `AVAudioPlayer` / `AVAudioEngine` — custom audio feedback
- `PHASE` (`PHASEEngine`, `PHASESoundEvent`) — spatial audio feedback in immersive spaces

**RealityKit**
- `CollisionComponent` — enable 3D hit testing on entities for direct touch and hover detection
- `InputTargetComponent` — make an entity respond to spatial tap gestures in RealityKit
- `HoverEffectComponent` — apply system hover highlight to a RealityKit entity (RealityKit equivalent of the SwiftUI `hoverEffect`)

## Code Highlights
3D tap gesture on a RealityKit entity via SwiftUI:

```swift
RealityView { content in
    let entity = ModelEntity(mesh: .generateSphere(radius: 0.1))
    entity.components.set(InputTargetComponent())
    entity.components.set(CollisionComponent(shapes: [.generateSphere(radius: 0.1)]))
    content.add(entity)
}
.onTapGesture {
    // handle tap
}
```

Spatial tap with 3D location:

```swift
.gesture(
    SpatialTapGesture()
        .targetedToAnyEntity()
        .onEnded { value in
            let location = value.location3D
            placeObject(at: location)
        }
)
```

## Takeaways
- Every interactive element must have a hover state — without one, users cannot confirm they are aiming correctly before committing a pinch.
- Design minimum touch targets at 60 pt (approximately 1 cm at arm's length in visionOS's default scale); smaller targets cause missed activations and user frustration.
- In fully immersive spaces with no visible controls, use an introductory coaching overlay for the first launch to teach custom gestures; do not assume discoverability.
- Add audio feedback for every significant interaction — vision is imprecise as a confirmation channel; sound provides the "click" that tells users an action registered.

---
_Source: WWDC24 Session 10096 page (abstract, chapter summaries, code samples, and resource links)._
