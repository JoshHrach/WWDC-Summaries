# Compose Interactive 3D Content in Reality Composer Pro
**WWDC24 · Session 10102** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10102/)

_Platforms:_ visionOS 2, iOS 18, macOS 15

## Overview
This session introduces the Timeline editor in Reality Composer Pro — a major new authoring feature that lets developers sequence 3D animations, audio, and interactions without writing code. Using a sample Botanist app with animated robots and plants, the presenter builds a complete interactive experience step by step, showing how timelines trigger from user gestures, nest inside each other, and integrate with RealityKit runtime APIs.

Beyond the visual editor, the session covers four new RealityKit animation APIs available in visionOS 2: Full Body Inverse Kinematics (IK), Animation Actions (built-in and custom), Blend Shape Weights, and Skeletal Poses. Together these systems let developers produce game-quality character animation that is driven by real-time data.

## Key Topics
- **Timelines** — the new Reality Composer Pro Timeline editor sequences built-in actions (Spin, Transform To, Play Animation, Play Audio, Notify) onto a track-based timeline with configurable durations and triggers.
- **Nested timelines** — a timeline can be inserted into another timeline, creating hierarchical animation compositions.
- **Behaviors component** — declaratively wires triggers (tap, collision, scene load, notification) to timelines without code; tap gestures still need a `.gesture` modifier in SwiftUI.
- **Inverse Kinematics** — `IKRig`, `IKResource`, and `IKComponent` expose Full Body IK; constrain individual joints with `.parent` and `.point` constraints; blend IK influence per constraint with `animationOverrideWeight`.
- **Animation Actions** — `EntityAction` protocol + `AnimationResource.makeActionAnimation` enable custom code-driven actions that sequence with built-in actions via `AnimationResource.sequence` and `AnimationResource.group`.
- **Blend Shapes** — `BlendShapeWeightsMapping`, `BlendShapeWeightsComponent`, and `BlendShapeWeightsSet` allow procedural weight updates to morph targets at runtime.
- **Skeletal Poses** — `SkeletalPosesComponent` provides per-joint transform access so a RealityKit `System` can rotate individual bones every frame.

## APIs & Frameworks

**RealityKit**
- `AnimationLibraryComponent` — store and associate USD animations with an entity; **[NEW]** exposed in Reality Composer Pro as a drag-and-drop target
- `AudioLibraryComponent` — store and associate audio resources with an entity; **[NEW]** exposed in Reality Composer Pro
- **[NEW]** `IKRig` — defines the IK solver configuration (skeleton, maxIterations, globalFkWeight); add constraints via `.constraints` array
- **[NEW]** `IKResource` — runtime data structure built from an `IKRig`; pass to `IKComponent`
- **[NEW]** `IKComponent` — attach to a model entity; configure constraints and update `animationOverrideWeight` each frame
- `IKRig.Constraint` — `.parent(named:on:positionWeight:orientationWeight:)`, `.point(named:on:positionWeight:)` constraint factories
- **[NEW]** `EntityAction` protocol — conform a struct to create a custom animation action; declare `animatedValueType`
- **[NEW]** `AnimationResource.makeActionAnimation(for:duration:)` — create an `AnimationResource` from a custom `EntityAction`
- `AnimationResource.sequence(with:)` — sequence multiple resources to play in order
- `AnimationResource.group(with:)` — play multiple resources simultaneously
- **[NEW]** `EntityAction.subscribe(to:_:)` — subscribe to `.started` / `.ended` events for a custom action type
- **[NEW]** `BlendShapeWeightsMapping` — create from a `MeshResource`; maps blend targets to weight indices
- **[NEW]** `BlendShapeWeightsComponent` — attach to entity; read/write `weightSet: BlendShapeWeightsSet`
- **[NEW]** `BlendShapeWeightsSet` — indexed collection of `BlendShapeWeights`; update `.weights` array to drive morph targets
- **[NEW]** `SkeletalPosesComponent` — already present on any skinned USD mesh; access `poses.default?.jointTransforms[index].rotation` to pose individual bones
- `Entity.playAnimation(_:)` — plays an `AnimationResource` on an entity; returns a `PlaybackController`
- `PlaybackController.stop()` — stop playback

**Reality Composer Pro (editor features, not runtime API)**
- Timeline editor with Spin, Transform To, Play Animation, Play Audio, Notify built-in actions
- Behaviors component (tap, collision, scene-added, notification triggers)
- AnimationLibraryComponent panel with clip slicing (scissors tool)
- Environment authoring and lighting tools (new in 2024)

## Code Highlights
Set up an IK rig and attach it to a character:

```swift
var rig = try IKRig(for: modelSkeleton)
rig.maxIterations = 30
rig.globalFkWeight = 0.02
rig.constraints = [
    .parent(named: "hips_constraint", on: hipsJointName,
            positionWeight: .init(repeating: 90), orientationWeight: .init(repeating: 90)),
    .point(named: "left_hand_constraint", on: leftHandJointName,
           positionWeight: .init(repeating: 10))
]
let resource = try IKResource(rig: rig)
entity.components.set(IKComponent(resource: resource))
```

Subscribe to a custom EntityAction ending:

```swift
struct RobotMoveToHomeComplete: EntityAction {
    var animatedValueType: (any AnimatableData.Type)? { nil }
}
RobotMoveToHomeComplete.subscribe(to: .ended) { event in
    // transition robot state
}
```

## Takeaways
- The Timeline editor eliminates boilerplate for simple sequenced animations; reserve `EntityAction` for logic that requires runtime data or custom state.
- IK constraints blend smoothly with existing USD animations via `animationOverrideWeight`; set it to 0 to let the base animation drive, 1 for full IK control.
- `SkeletalPosesComponent` is automatically present on any rigged USD import — no setup required, just update joint transforms inside a RealityKit `System.update`.
- Use `AnimationLibraryComponent` clip slicing in Reality Composer Pro to break a single long USD animation into reusable named clips.

---
_Source: WWDC24 Session 10102 page (abstract, chapter summaries, code samples, and resource links)._
