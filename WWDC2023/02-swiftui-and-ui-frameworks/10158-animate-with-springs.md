# Animate with Springs
**WWDC23 · Session 10158** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10158/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1

## Overview
This session makes a deep case for spring animations as the best general-purpose animation tool for Apple platform UIs, then explains the physics behind them and shows the new unified API available in SwiftUI, UIKit, and Core Animation. Springs are now the **default animation** in SwiftUI — calling `withAnimation {}` without arguments uses a spring.

The key insight is that springs are uniquely capable of maintaining continuous position and velocity, which makes them the only animation type that feels natural both in static transitions and in gesture-driven scenarios where the initial velocity must be preserved. Springs don't need to bounce to be useful; the smooth, gradual "long tail" approach to rest is what makes non-bouncy springs feel physical and polished.

A new `Spring` model type in SwiftUI exposes spring mathematics programmatically, allowing parameter conversion, evaluation at arbitrary time points, and construction of custom animations.

## Key Topics

### Why Springs
- **Continuity of position and velocity** — unlike linear animations (which have velocity jumps) or ease-in/out curves (which can't represent initial velocity from a gesture), springs handle both.
- **Gesture hand-off** — SwiftUI now automatically tracks gesture velocities and passes them into spring animations; no extra code needed.
- **Interruption handling** — when a new animation begins before the previous one finishes, the spring uses the current velocity as its new initial velocity, making interruptions feel seamless.

### How Springs Work (Physics)
- Spring motion modeled by three physical parameters: **mass**, **stiffness**, **damping**.
- New simplified configuration uses just two parameters: **duration** (perceptual, not settling) and **bounce** (−1.0 to 1.0).
- Three spring types:
  - **Bouncy** (bounce > 0): damped cosine wave — overshoots target
  - **Smooth** (bounce = 0): line × exponential decay — gradual approach
  - **Flattened** (bounce < 0): two exponentials — even slower approach; useful for scroll deceleration
- **Settling duration** vs **duration parameter**: settling duration is when the animation physically ends (unpredictable); duration is a perceptual tuning parameter (stable). Use SwiftUI's completion handler (which uses perceptual duration) for post-animation UI changes.

### How to Use Springs
- Spring is now the **default SwiftUI animation** (`withAnimation { }` uses `.smooth` spring preset by default)
- Three built-in spring presets based on iOS system animations: `.smooth`, `.snappy`, `.bouncy`
- Presets are tunable with `duration:` and `extraBounce:` parameters
- Fully custom springs via `.spring(duration:bounce:)`
- `Spring` model type for programmatic math

## APIs & Frameworks

### SwiftUI
- `withAnimation { }` — now defaults to spring animation **[NEW default]**
- `Animation.spring(duration:bounce:)` — custom spring with duration (seconds) and bounce (−1.0 to 1.0) **[NEW signature]**
- `Animation.smooth` — built-in preset (smooth, no bounce) **[NEW]**
- `Animation.smooth(duration:extraBounce:)` — tunable smooth preset **[NEW]**
- `Animation.snappy` — built-in preset (slightly brisk feel) **[NEW]**
- `Animation.snappy(duration:extraBounce:)` **[NEW]**
- `Animation.bouncy` — built-in preset (noticeable bounce) **[NEW]**
- `Animation.bouncy(duration:extraBounce:)` **[NEW]**
- `Spring` — new model type representing a spring configuration **[NEW]**
  - `Spring(duration:bounce:)` — init from perceptual parameters
  - `Spring(mass:stiffness:damping:)` — init from physical parameters
  - `spring.mass`, `spring.stiffness`, `spring.damping` — read computed physical parameters
  - `spring.value(target:time:)` — evaluate position at a given time
  - `spring.velocity(target:time:)` — evaluate velocity at a given time
  - `spring.value(target:initialVelocity:time:)` — evaluate with initial velocity
- `Animation.spring(_:)` — use a `Spring` model as an animation directly **[NEW]**
- `AnimationContext` — used in custom `Animatable` to access `initialVelocity`
- Completion handler support in SwiftUI spring animations (uses perceptual duration) **[NEW]**

### UIKit
- `UIView.animate(duration:bounce:animations:)` — new UIKit spring animation method using perceptual duration/bounce parameters **[NEW]**

### Core Animation
- `CASpringAnimation(perceptualDuration:bounce:)` — new `CASpringAnimation` initializer using the unified duration/bounce model **[NEW]**

### Parameter Conversion (mathematical reference)
```
mass = 1
stiffness = (2π ÷ duration)²
damping = (1 - 4π × bounce ÷ duration)  [bounce ≥ 0]
          (4π ÷ (duration + 4π × bounce)) [bounce < 0]
```

## Code Highlights

Spring presets and custom springs:
```swift
// Default spring (smooth preset)
withAnimation { isActive.toggle() }

// Named preset, tuned
withAnimation(.snappy(duration: 0.4, extraBounce: 0.1)) { ... }

// Fully custom spring
withAnimation(.spring(duration: 0.6, bounce: 0.2)) { ... }

// UIKit
UIView.animate(duration: 0.6, bounce: 0.2) { ... }

// Core Animation
let animation = CASpringAnimation(perceptualDuration: 0.6, bounce: 0.2)
```

Using the Spring model to evaluate position over time:
```swift
let mySpring = Spring(duration: 0.4, bounce: 0.2)
let value = mySpring.value(target: 1, time: time)
let velocity = mySpring.velocity(target: 1, time: time)
```

## Takeaways
- Springs are now the default SwiftUI animation — lean on `.smooth`, `.snappy`, `.bouncy` presets before reaching for custom values.
- A spring with bounce 0 (smooth) is the best general-purpose choice; only add bounce when the animation should feel playful or physically gesture-like.
- The new `Spring` model type makes spring math accessible for custom animations, simulations, and chart data generation.
- `UIView.animate(duration:bounce:)` and `CASpringAnimation(perceptualDuration:bounce:)` bring the unified parameter model to UIKit and Core Animation.

---
_Source: WWDC23 Session 10158 page (abstract, chapter summaries, code samples, and resource links)._
