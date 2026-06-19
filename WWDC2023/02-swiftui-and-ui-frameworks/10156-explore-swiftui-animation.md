# Explore SwiftUI Animation
**WWDC23 · Session 10156** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10156/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, watchOS 10, visionOS 1

## Overview
This session is a deep technical dive into SwiftUI's animation system, explaining how its three core primitives — `Animatable`, `Animation`, and `Transaction` — work together to produce visual effects. It covers the full anatomy of a view update (dependency tracking, attribute graph invalidation, body calls), how animatable attributes interpolate presentation values from a transaction's animation, and all the new additions in iOS 17: `CustomAnimation` protocol, `UnitCurve` standalone model, new `Spring` initializer, `TransactionKey` for custom transaction data, scoped `.animation(_:body:)` modifier, and scoped `.transaction(_:body:)` modifier.

The session emphasizes that `withAnimation` now defaults to a smooth spring in iOS 17, and explains shouldMerge/velocity requirements on `CustomAnimation` for handling interrupted animations and velocity preservation.

## Key Topics

- **Anatomy of a view update** — SwiftUI maintains a persistent attribute graph; state changes open a transaction, invalidate downstream attributes, call body, and update the graph. Animatable attributes have both a model value and a presentation value, and interpolate between them using the transaction's animation.
- **Animatable protocol** — Views conforming to `Animatable` define `animatableData: VectorArithmetic` (e.g., `CGFloat`, `CGPoint`, `AnimatablePair`); SwiftUI builds an animatable attribute whose body is called every frame, enabling effects like custom layout animations along an arc. Use sparingly — custom `Animatable` is expensive (calls body per frame).
- **Animation categories** — Three built-in categories:
  - **Timing curve** — `Animation.linear`, `.easeIn`, `.easeOut`, `.easeInOut`; configurable duration; defined by bezier control points via `UnitCurve`.
  - **Spring** — new intuitive `Spring(duration:bounce:)` initializer; presets: `.smooth`, `.snappy`, `.bouncy`; preserves velocity via `shouldMerge`; **now the default** for bare `withAnimation` in iOS 17.
  - **Higher-order** — `.delay()`, `.speed()`, `.repeatCount()`, `.repeatForever(autoreverses:)`.
- **CustomAnimation protocol [NEW]** — Define completely custom interpolation; required method: `animate(value:time:context:) -> V?`; optional: `shouldMerge`, `velocity`; operates on delta vectors (VectorArithmetic).
- **UnitCurve [NEW]** — Standalone bezier curve model; `UnitCurve(startControlPoint:endControlPoint:)`; `value(at:)` and `velocity(at:)` methods for use outside of animation contexts.
- **Spring model [NEW]** — Standalone spring simulation; `Spring(duration:bounce:)`; `value(target:time:)` and `velocity(target:time:)` methods.
- **Transaction** — Implicit dictionary propagated through the attribute graph during an update; carries the current animation and any custom keys. Discarded after every update. APIs: `withAnimation`, `withTransaction(_:_:)`, `.transaction(value:_:)`, `.transaction(_:body:)`.
- **TransactionKey [NEW]** — Protocol for defining custom transaction dictionary keys (analogous to `EnvironmentKey`); `defaultValue` required; computed property on `Transaction` extension for access; set via `withTransaction(\.myKey, value) { }`.
- **Scoped animation modifier [NEW]** — `.animation(_:body:)` — narrows the animation to only the animatable attributes specified in the body closure; prevents accidental animation of child content; critical for reusable/generic view components.
- **Scoped transaction modifier [NEW]** — `.transaction(_:body:)` — narrows transaction modifications to a specific sub-hierarchy; analogous to scoped animation; two variants: `transaction(value:_:)` and `transaction(_:body:)`.
- **shouldMerge and velocity in CustomAnimation** — `shouldMerge` determines how a new animation combines with a currently-running one (timing curves: additive; springs: merge and retarget preserving velocity); `velocity` enables velocity hand-off when animations are merged.

## APIs & Frameworks

**SwiftUI — Animation**
- `CustomAnimation` protocol **[NEW]** — implement custom animation algorithms
- `CustomAnimation.animate(value:time:context:)` **[NEW]** — required; returns current interpolated value or nil when finished
- `CustomAnimation.shouldMerge(previous:value:time:context:)` **[NEW]** — optional; controls how new animation merges with running one
- `CustomAnimation.velocity(value:time:context:)` **[NEW]** — optional; provides velocity for hand-off on merge
- `AnimationContext` **[NEW]** — passed to `CustomAnimation` methods; holds state for the animation
- `Animation.animate(using:)` **[NEW]** — wraps a `CustomAnimation` instance as `Animation`
- `UnitCurve` **[NEW]** — standalone bezier curve; `UnitCurve(startControlPoint:endControlPoint:)`, `value(at:)`, `velocity(at:)`
- `Spring` **[NEW]** — standalone spring model; `Spring(duration:bounce:)`, `value(target:time:)`, `velocity(target:time:)`
- `Animation.smooth` **[NEW preset]** — smooth spring (no bounce); new default for `withAnimation` in iOS 17
- `Animation.snappy` **[NEW preset]** — spring with small bounce
- `Animation.bouncy` **[NEW preset]** — spring with larger bounce
- `Animation.smooth(duration:extraBounce:)` **[NEW]** — tunable smooth spring
- `Animation.snappy(duration:extraBounce:)` **[NEW]** — tunable snappy spring
- `Animation.bouncy(duration:extraBounce:)` **[NEW]** — tunable bouncy spring
- `withAnimation` — now defaults to `.smooth` (spring) in iOS 17 **[behavior change]**
- `withTransaction(_:_:)` **[existing]** — open transaction with custom transaction mutations

**SwiftUI — Transaction**
- `TransactionKey` protocol **[NEW]** — define custom keys for the transaction dictionary; requires `defaultValue`
- `Transaction` — existing; new subscript access via custom `TransactionKey`
- `.transaction(value:_:)` **[NEW]** — scoped transaction modifier; only modifies transaction when `value` changes
- `.transaction(_:body:)` **[NEW]** — scoped transaction modifier; narrowly applies to the specified sub-hierarchy

**SwiftUI — Animatable**
- `Animatable` protocol — existing; `animatableData` property of type `VectorArithmetic`
- `VectorArithmetic` — existing; `scaled(by:)`, vector addition/subtraction
- `AnimatablePair` — existing; fuses two `VectorArithmetic` vectors into one larger vector
- `scaleEffect` — existing animatable modifier; `animatableData` is `AnimatablePair<CGSize, UnitPoint>` (four-dimensional)

**SwiftUI — View Modifiers**
- `.animation(_:value:)` — existing; write animation into transaction only when `value` changes
- `.animation(_:body:)` **[NEW]** — scoped animation modifier; limits animation to attributes in body closure; prevents accidental child animation
- `.transaction(value:_:)` **[NEW]** — scoped transaction modifier; limits to when value changes
- `.transaction(_:body:)` **[NEW]** — scoped transaction modifier body variant

## Code Highlights

Custom animation implementation (linear timing curve):
```swift
struct MyLinearAnimation: CustomAnimation {
    var duration: TimeInterval

    func animate<V: VectorArithmetic>(value: V, time: TimeInterval,
                                      context: inout AnimationContext<V>) -> V? {
        if time <= duration {
            value.scaled(by: time / duration)
        } else {
            nil // animation has finished
        }
    }

    func velocity<V: VectorArithmetic>(value: V, time: TimeInterval,
                                       context: AnimationContext<V>) -> V? {
        value.scaled(by: 1.0 / duration)
    }
}
```

Scoped animation to prevent accidental child animations:
```swift
struct Avatar<Content: View>: View {
    var content: Content
    @Binding var selected: Bool

    var body: some View {
        content
            .animation(.smooth) { $0.shadow(radius: selected ? 12 : 8) }
            .animation(.bouncy) { $0.scaleEffect(selected ? 1.5 : 1.0) }
            .onTapGesture { selected.toggle() }
    }
}
```

Custom TransactionKey for context-aware animation selection:
```swift
private struct AvatarTappedKey: TransactionKey {
    static let defaultValue = false
}
extension Transaction {
    var avatarTapped: Bool {
        get { self[AvatarTappedKey.self] }
        set { self[AvatarTappedKey.self] = newValue }
    }
}

// Usage:
withTransaction(\.avatarTapped, true) {
    selected.toggle()
}
```

## Takeaways

- `withAnimation` now defaults to a smooth spring in iOS 17 — use `.bouncy` or `.snappy` presets for interactive gestures where organic feel matters, and smooth for programmatic changes.
- Use `.animation(_:body:)` (scoped animation modifier) instead of `.animation(_:value:)` in reusable generic components; it narrows the animation strictly to the animatable attributes you list, preventing accidental animation of arbitrary child content.
- `CustomAnimation` lets you implement completely custom interpolation at the same generic level as built-in animations; operate on delta vectors (not absolute values) and return `nil` when finished.
- `TransactionKey` enables implicitly propagating update-specific context (e.g., whether a change was interactive vs. programmatic) through the transaction dictionary, allowing views deep in the hierarchy to make informed animation decisions without explicit parameters.

---
_Source: WWDC23 Session 10156 page (abstract, chapter summaries, code samples, and resource links)._
