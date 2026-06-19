# Wind your way through advanced animations in SwiftUI
**WWDC23 · Session 10157** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10157/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1

## Overview
SwiftUI iOS 17 introduces two powerful new families of animation API: `PhaseAnimator` for declarative multi-step animations that advance automatically or in response to events, and `KeyframeAnimator` / `KeyframeTimeline` for frame-accurate, per-property animation tracks with four interpolation modes (linear, spring, cubic, and move). Together they make repeating ambient animations and event-driven celebratory effects straightforward, while giving advanced users a way to drive any SwiftUI modifier — or even MapKit camera — with precisely crafted curves.

## Key Topics

**Animation Phases (`PhaseAnimator`)**
- `PhaseAnimator` replaces the pattern of toggling state + `.animation` for multi-step sequences
- Provide an array (or `CaseIterable` enum) of phases; SwiftUI advances through them automatically, looping forever — perfect for ambient/repeating effects
- Trigger variant: add `trigger:` parameter; SwiftUI animates through phases once every time the trigger value changes — perfect for event-driven effects (e.g., someone liked a post)
- Customize per-phase animation via trailing `animation:` closure; the destination phase is passed in so each transition can use a different curve

**Keyframes (`KeyframeAnimator`)**
- Unlike phases (one value for all properties at once), keyframes animate each property independently in parallel tracks
- Define an `Animatable`-conforming struct holding all animated properties; `Double` now conforms to `Animatable` **[NEW]**
- `KeyframeTrack` specifies which property to animate (via key path) and lists keyframes
- Four keyframe types, freely mixed within a track:
  - `LinearKeyframe` — linear interpolation in vector space
  - `SpringKeyframe` — spring function toward the target; optional `duration` caps the spring
  - `CubicKeyframe` — cubic Bézier; adjacent cubic keyframes form a Catmull-Rom spline
  - `MoveKeyframe` — immediate jump, no interpolation
- SwiftUI maintains velocity across keyframe boundaries for continuous motion
- Because keyframes are "predefined animations" (like video clips), they don't gracefully retarget — avoid changing keyframes mid-animation

**MapKit Camera Keyframes (NEW)**
- `mapCameraKeyframeAnimation(trigger:keyframes:)` modifier **[NEW]** on `Map`
- Provides `initialCamera: MapCamera` to keyframe closure; tracks animate `\.centerCoordinate`, `\.heading`, `\.distance`
- User gestures interrupt and take over; final keyframe value becomes the resting camera

**`KeyframeTimeline` — manual evaluation**
- Capture keyframe tracks outside a view: `KeyframeTimeline(initialValue:) { tracks }`
- `.duration` — total duration (longest track)
- `.value(time:)` — compute value at any time for use in Swift Charts, `TimelineView`, geometry-driven scroll effects, etc.

## APIs & Frameworks

**SwiftUI — PhaseAnimator**
- `.phaseAnimator(_:content:animation:)` modifier **[NEW]** — repeating animation through provided phases
- `.phaseAnimator(_:trigger:content:animation:)` modifier **[NEW]** — trigger-driven animation

**SwiftUI — KeyframeAnimator**
- `.keyframeAnimator(initialValue:trigger:content:keyframes:)` modifier **[NEW]**
- `KeyframeTrack<Root, Value>` **[NEW]** — a single-property animation track identified by key path
- `LinearKeyframe(_:duration:timingCurve:)` **[NEW]**
- `SpringKeyframe(_:duration:spring:)` **[NEW]**
- `CubicKeyframe(_:duration:)` **[NEW]**
- `MoveKeyframe(_:)` **[NEW]**
- `KeyframeTimeline` **[NEW]** — standalone keyframe container for manual evaluation
- `KeyframeTimeline.duration` **[NEW]**
- `KeyframeTimeline.value(time:)` **[NEW]**
- `Double: Animatable` conformance **[NEW]**

**MapKit**
- `Map.mapCameraKeyframeAnimation(trigger:keyframes:)` **[NEW]**
- `MapCamera` — keyframe-animatable type; properties: `\.centerCoordinate`, `\.heading`, `\.distance`

## Code Highlights

Repeating phase animation (ambient reminder pulse):
```swift
OverdueReminderView()
    .phaseAnimator([false, true]) { content, value in
        content
            .foregroundStyle(value ? .red : .primary)
    } animation: { _ in
        .easeInOut(duration: 1.0)
    }
```

Event-triggered phase animation (reaction badge):
```swift
enum Phase: CaseIterable {
    case initial, move, scale

    var verticalOffset: Double {
        switch self { case .initial: 0; case .move, .scale: -64 }
    }
    var scale: Double {
        switch self { case .initial: 1.0; case .move: 1.1; case .scale: 1.8 }
    }
}

ReactionView()
    .phaseAnimator(Phase.allCases, trigger: reactionCount) { content, phase in
        content
            .scaleEffect(phase.scale)
            .offset(y: phase.verticalOffset)
    } animation: { phase in
        switch phase {
        case .initial: .smooth
        case .move: .easeInOut(duration: 0.3)
        case .scale: .spring(duration: 0.3, bounce: 0.7)
        }
    }
```

Multi-track keyframe animation (celebration heart icon):
```swift
struct AnimationValues {
    var scale = 1.0
    var verticalStretch = 1.0
    var verticalTranslation = 0.0
    var angle = Angle.zero
}

ReactionView()
    .keyframeAnimator(initialValue: AnimationValues()) { content, value in
        content
            .rotationEffect(value.angle)
            .scaleEffect(value.scale)
            .scaleEffect(y: value.verticalStretch)
            .offset(y: value.verticalTranslation)
    } keyframes: { _ in
        KeyframeTrack(\.scale) {
            LinearKeyframe(1.0, duration: 0.36)
            SpringKeyframe(1.5, duration: 0.8, spring: .bouncy)
            SpringKeyframe(1.0, spring: .bouncy)
        }
        KeyframeTrack(\.verticalTranslation) {
            LinearKeyframe(0.0, duration: 0.1)
            SpringKeyframe(20.0, duration: 0.15, spring: .bouncy)
            SpringKeyframe(-60.0, duration: 1.0, spring: .bouncy)
            SpringKeyframe(0.0, spring: .bouncy)
        }
        // ... angle, verticalStretch tracks omitted for brevity
    }
```

MapKit camera tour:
```swift
Map(initialPosition: .rect(route.rect)) { ... }
    .mapCameraKeyframeAnimation(trigger: playTrigger) { initialCamera in
        KeyframeTrack(\MapCamera.centerCoordinate) {
            for point in route.points {
                CubicKeyframe(point.coordinate, duration: 16.0 / Double(route.points.count))
            }
            CubicKeyframe(initialCamera.centerCoordinate, duration: 4.0)
        }
        KeyframeTrack(\.distance) {
            CubicKeyframe(24000, duration: 4)
            CubicKeyframe(18000, duration: 12)
            CubicKeyframe(initialCamera.distance, duration: 4)
        }
    }
```

## Takeaways
- Use `PhaseAnimator` with a simple `[false, true]` array for looping ambient effects — it replaces the manual toggle + `.animation` pattern with far less boilerplate.
- Add the `trigger:` parameter to `PhaseAnimator` to play an animation sequence exactly once per event (like a reaction or notification), then return to the resting state automatically.
- Reserve `KeyframeAnimator` for celebratory or cinematic moments where you need independent control over each property: squash/stretch, rotation, and translation with different curves per track make animations feel physical and alive.
- Use `KeyframeTimeline` to manually evaluate keyframe curves for scroll-driven effects, Swift Charts visualizations, or `TimelineView`-based playback.

---
_Source: WWDC23 Session 10157 page (abstract, chapter summaries, transcript, and code samples)._
