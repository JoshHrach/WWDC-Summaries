# Enhance Your UI Animations and Transitions
**WWDC24 · Session 10145** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10145/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, tvOS 18, visionOS 2

## Overview
iOS 18 and SwiftUI add a new zoom navigation transition and a richer suite of UIKit spring animations. The session covers four independent but complementary topics: the SwiftUI zoom transition (and its UIKit counterpart), new UIKit `UIView.animate` spring API, `context.animate { }` for bridging UIKit animations into SwiftUI's update cycle, and interactive/interruptible spring animations driven by gestures.

Together these APIs close the gap between SwiftUI and UIKit animation expressiveness and make gesture-driven, physics-feeling interactions much easier to build without custom `UIViewPropertyAnimator` scaffolding.

## Key Topics

### Zoom Navigation Transition
SwiftUI's new `navigationTransitionStyle(.zoom(sourceID:in:))` on a `NavigationLink` destination, paired with `matchedTransitionSource(id:in:)` on the source view, creates a hero-zoom animation where the destination view expands from the tapped source. The namespace and ID system mirrors `matchedGeometryEffect`; the animation is interruptible.

UIKit equivalent: `viewController.preferredTransition = .zoom { context in context.zoomedView }` on the pushed or presented view controller, where the closure returns the source view to zoom from.

### UIKit Spring Animations
`UIView.animate(.spring(duration:))` replaces the old `UIView.animate(withDuration:delay:usingSpringWithDamping:...)` variant. The spring is physics-based; you can also pass `.spring(bounce:)` for a bounciness-based shorthand. Completion handlers and keyframe animations work identically to the old API.

### Bridging UIKit Animations into SwiftUI
When a `UIViewRepresentable` updates via `updateUIView(_:context:)`, changes should animate in sync with the enclosing SwiftUI transaction. `context.animate { }` inside `updateUIView` wraps imperative UIKit changes in the current SwiftUI animation, so UIKit views animate alongside their SwiftUI siblings without any explicit `UIViewPropertyAnimator`.

### Interactive/Gesture-Driven Springs
`UIView.animate(.interactiveSpring)` returns a `UIViewPropertyAnimator` configured with a spring that can be driven by a gesture recognizer's `fractionComplete`. Calling `continueAnimation(withTimingParameters:durationFactor:)` when the gesture ends lets the spring finish naturally. This replaces manual spring simulation for swipe-to-dismiss and swipe-to-expand patterns.

## APIs & Frameworks

**SwiftUI**
- `navigationTransitionStyle(.zoom(sourceID:in:))` on `NavigationLink` destination **[NEW]**
- `matchedTransitionSource(id:in:)` modifier **[NEW]**
- `Namespace` (existing, used with zoom transition)

**UIKit**
- `UIViewController.preferredTransition: UIViewController.Transition` **[NEW]**
  - `.zoom { context -> UIView in ... }` **[NEW]**
- `UIView.animate(_:body:completion:)` **[NEW overload]**
  - `.spring(duration:bounce:)` **[NEW]** — physics-based duration + bounciness parameter
  - `.spring(duration:)` **[NEW]**
  - `.interactiveSpring` **[NEW]** — gesture-drivable spring returning `UIViewPropertyAnimator`
- `UIViewPropertyAnimator` (existing, returned by `.interactiveSpring`)
  - `.continueAnimation(withTimingParameters:durationFactor:)` (existing, commonly used to finish interactive springs)

**SwiftUI–UIKit interop**
- `UIViewRepresentableContext.animate { }` **[NEW]** — wraps UIKit updates in the current SwiftUI transaction animation

## Code Highlights

```swift
// SwiftUI zoom transition
@Namespace var zoom

NavigationLink {
    DetailView()
        .navigationTransitionStyle(.zoom(sourceID: item.id, in: zoom))
} label: {
    ThumbnailView(item: item)
        .matchedTransitionSource(id: item.id, in: zoom)
}

// UIKit zoom transition
destination.preferredTransition = .zoom { context in
    return sourceCellView  // the view to zoom from
}

// New spring animation
UIView.animate(.spring(duration: 0.5, bounce: 0.3)) {
    myView.transform = CGAffineTransform(scaleX: 1.5, y: 1.5)
}

// Bridging UIKit into SwiftUI animation
func updateUIView(_ uiView: MyView, context: Context) {
    context.animate {
        uiView.backgroundColor = isSelected ? .systemBlue : .systemGray
    }
}

// Interactive spring
let animator = UIView.animate(.interactiveSpring) {
    draggableView.transform = CGAffineTransform(translationX: 0, y: 300)
}
// In pan gesture recognizer:
animator.fractionComplete = recognizer.translation(in: view).y / 300
// On gesture end:
animator.continueAnimation(withTimingParameters: nil, durationFactor: 0)
```

## Takeaways
- Use `matchedTransitionSource` + `navigationTransitionStyle(.zoom)` for hero zoom navigation — no custom `UIViewControllerTransitioningDelegate` required.
- Replace `usingSpringWithDamping:initialSpringVelocity:` with `UIView.animate(.spring(duration:bounce:))` for cleaner spring declarations and better physics.
- Call `context.animate { }` inside `updateUIView` to ensure UIKit subviews animate in sync with SwiftUI state changes.
- For gesture-driven dismissals, use `.interactiveSpring` and drive `fractionComplete` — the spring naturally continues to its end state when the gesture completes.

---
_Source: WWDC24 Session 10145 page (abstract, chapter summaries, code samples, and resource links)._
