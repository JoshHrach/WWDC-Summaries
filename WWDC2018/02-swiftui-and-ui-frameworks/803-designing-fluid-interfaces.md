# Designing Fluid Interfaces
**WWDC18 · Session 803** · [Watch](https://developer.apple.com/videos/play/wwdc2018/803/)

_Platforms:_ iOS, iPadOS

## Overview
This session deconstructs the design philosophy behind the fluid, gesture-driven interfaces introduced with iPhone X. Presenters from the Apple Human Interface and iOS Engineering teams explain how gestures and animations must feel immediate, continuous, and interruptible to feel "fluid." The session analyzes the physics of spring animations, the math of rubber-band scrolling, techniques for building interruptible animations, and design heuristics for gesture-driven interactions — all with live code demonstrations in a playground.

The core thesis: interfaces feel fluid when control is transferred directly from the user's finger to the content with minimal latency, and when stopping or reversing an action is always possible without jarring transitions.

## Key Topics

### What Makes an Interface Feel Fluid

- **Immediate response** — UI reacts the instant the touch begins, not after a tap is confirmed. Delay of even 100ms feels laggy.
- **Continuous response** — the UI tracks the user's finger 1:1 throughout the gesture; content never jumps or teleports.
- **Interruptible animations** — any in-progress animation can be grabbed mid-flight and redirected. A non-interruptible animation that ignores a new gesture creates user frustration.
- **Maintain momentum** — when a user flicks, the velocity of their finger is preserved into the animation that follows; abrupt stops break the physical intuition.

### Spring Animations

- Springs are the foundation of iOS physics. A spring described by **stiffness** and **damping** is more physically correct than a duration + curve.
- **Critically damped springs** — reach their target without oscillating; feel "snappy" while preserving velocity continuity. Used for most iOS system animations (app launch, sheet presentation).
- **Underdamped springs** — oscillate slightly past the target and settle; useful for conveying a "bouncy" or emphatic feel (e.g., icon shake, watch complication select).
- **`UISpringTimingParameters`** — construct with `dampingRatio` (0–1; 1 = critical) and optionally `initialVelocity` (a `CGVector`) to carry finger velocity into the spring.
- **`UIViewPropertyAnimator`** with spring timing — interruptible by default; calling `pauseAnimation()` freezes it, then `continueAnimation(withTimingParameters:durationFactor:)` restarts from current state with new velocity.

### Rubber-Band Scrolling

- When content is scrolled past its boundary, resistance is applied: displacement = `x - (x * limit) / (x + limit)`.
- This formula always converges toward `limit` but never reaches it — creates the characteristic "harder to pull" feel.
- The same math can be applied to any constrained drag (card lift, pull-to-refresh tension).

### Gesture-Driven Animations (Interactive Transitions)

- Connect a `UIPanGestureRecognizer` directly to a `UIViewPropertyAnimator`.
- On `.began`: create and immediately pause the animator.
- On `.changed`: update `fractionComplete` proportionally to finger displacement relative to total travel distance.
- On `.ended`: read `gestureRecognizer.velocity(in:)`, project the finish position, decide commit/cancel, then call `continueAnimation(withTimingParameters:durationFactor:)` with a spring carrying the finger velocity.
- **Additive animations** — when redirecting an in-flight animation, the new animation adds to the current position/velocity rather than jumping to a new start. Use `isAdditive = true` on `CABasicAnimation` layers for continuous feel.
- Avoid `UIView.animate(withDuration:)` for interactive content — it is not interruptible.

### Design Heuristics

- **Hit targets** — make touch targets significantly larger than their visual representation; the gap between icon and touch boundary should feel invisible.
- **Interrupt, don't ignore** — never discard a gesture while an animation is running. Always allow the user to take control.
- **Direct manipulation** — objects should follow the touch point, not animate toward it independently.
- **Match deceleration to context** — a fast scroll deceleration feels jarring inside a picker; a slow deceleration in a full-screen dismiss feels sluggish.
- **Use velocity projection** — when a gesture ends, project where the content would travel if released at that velocity and use that to decide commit vs. cancel.

## APIs & Frameworks

**UIKit**
- `UIViewPropertyAnimator` — `init(duration:timingParameters:)`, `pauseAnimation()`, `continueAnimation(withTimingParameters:durationFactor:)`, `fractionComplete`, `isRunning`, `isReversed`
- `UISpringTimingParameters` — `init(dampingRatio:initialVelocity:)` — key: pass finger velocity as `CGVector` for momentum continuity
- `UICubicTimingParameters` — for non-spring curves
- `UIPanGestureRecognizer` — `velocity(in:)` — read on `.ended` to seed spring `initialVelocity`
- `UIViewAnimating` protocol — `pauseAnimation()`, `stopAnimation(_:)`, `finishAnimation(at:)`
- `UIViewImplicitlyAnimating` protocol — `addAnimations(_:)`, `addCompletion(_:)`

**Core Animation**
- `CASpringAnimation` — `stiffness`, `damping`, `initialVelocity`, `mass`, `settlingDuration`
- `CABasicAnimation` — `isAdditive` — enable additive behavior for mid-flight redirection
- `CADisplayLink` — drive manual physics simulations at display refresh rate

## Code Highlights

Building a pan-gesture-driven interactive animator with velocity hand-off:
```swift
var animator: UIViewPropertyAnimator?

@objc func handlePan(_ recognizer: UIPanGestureRecognizer) {
    switch recognizer.state {
    case .began:
        animator = UIViewPropertyAnimator(duration: 0.5,
            timingParameters: UISpringTimingParameters(dampingRatio: 1.0))
        animator?.addAnimations { self.card.transform = .identity }
        animator?.pauseAnimation()

    case .changed:
        let translation = recognizer.translation(in: view)
        animator?.fractionComplete = translation.y / totalDistance

    case .ended:
        let velocity = recognizer.velocity(in: view)
        let springVelocity = CGVector(dx: 0, dy: velocity.y / totalDistance)
        let params = UISpringTimingParameters(dampingRatio: 1.0,
                                             initialVelocity: springVelocity)
        animator?.continueAnimation(withTimingParameters: params, durationFactor: 0)

    default: break
    }
}
```

Rubber-band displacement formula:
```swift
func rubberBand(offset: CGFloat, dimension: CGFloat, rate: CGFloat = 0.55) -> CGFloat {
    return (1.0 - (1.0 / (offset * rate / dimension + 1.0))) * dimension
}
```

## Takeaways
- Fluid interfaces are built on three pillars: immediate response, 1:1 tracking, and interruptibility — any one missing breaks the physical intuition.
- `UIViewPropertyAnimator` + `UISpringTimingParameters(dampingRatio:initialVelocity:)` is the correct pair for interactive, interruptible animations in UIKit; always seed `initialVelocity` from the gesture recognizer's `velocity(in:)`.
- Spring parameters (stiffness/damping) are more meaningful than duration + easing curve because they naturally handle variable initial velocities.
- Rubber-band math (`x - (x * limit) / (x + limit)`) applies anywhere a drag needs to feel increasingly difficult as it approaches a boundary.

---
_Source: WWDC18 Session 803 page (abstract, full transcript, and resource links)._
