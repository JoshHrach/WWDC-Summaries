# Explore Concurrency in SwiftUI
**WWDC25 · Session 266** · [Watch](https://developer.apple.com/videos/play/wwdc2025/266/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, tvOS 26, watchOS 26, visionOS 26

## Overview
This session explains how SwiftUI's concurrency model works end-to-end: why SwiftUI defaults to `@MainActor` isolation for all views and what that means for model types, how SwiftUI offloads selected work to background threads (and which APIs trigger this), and how to structure app code so that UI updates remain synchronous while long-running work runs off-thread without data races.

It is the conceptual companion to "Embracing Swift Concurrency" (Session 268) and the code-along "Elevate an App with Swift Concurrency" (Session 270).

## Key Topics

### Main Actor by Default
- `View` protocol is declared `@MainActor`, so every struct conforming to `View` is implicitly `@MainActor` isolated.
- All `View` members (`body`, `@State` properties, action closures) are also implicitly `@MainActor`.
- SwiftUI models instantiated inside a `View` inherit the view's actor isolation automatically — no explicit `@MainActor` annotation needed on the model.
- `UIViewRepresentable` refines `View`, so all its methods (e.g., `makeUIView`) are also `@MainActor` isolated; UIKit/AppKit APIs that require `@MainActor` work seamlessly.
- **Swift 6.2** — introduces implicit `@MainActor` for all types in a module (new language mode). Existing apps can adopt this to delete most explicit `@MainActor` annotations.

### SwiftUI's Background Thread Optimizations
- SwiftUI runs certain computationally expensive callbacks on background threads:
  - `Shape.path(in:)` — called from a background thread during animation.
  - `visualEffect` closure — called from background thread; capture only value types.
  - `Layout` protocol methods (`sizeThatFits`, `placeSubviews`) — may be called off-main.
  - `onGeometryChange` closure — may be called off-main.
- These APIs are annotated with `Sendable` in SwiftUI's type system to communicate the cross-isolation boundary to the compiler.
- The compiler will emit an error if you access a `@MainActor`-isolated property from a `Sendable` closure — fix by capturing a copy of the value in the closure's capture list.

### Fixing Sendable Errors in Closures
- Accessing `self.someProperty` (a `@MainActor`-isolated member) inside a `Sendable` closure is a compile error.
- **Fix**: capture the value explicitly: `{ [propertyValue = self.someProperty] in ... }`.
- This sends a copy of the value (e.g., a `Bool`) rather than `self`, avoiding actor isolation violations.
- Alternative: make the accessed property `nonisolated` if it's safe to access without actor protection.

### Synchronous vs. Asynchronous UI Updates
- SwiftUI action callbacks (`Button` action, `onTapGesture`, etc.) are **synchronous by design**.
- Time-sensitive state mutations for animations should happen synchronously (`withAnimation { state = newValue }`), before any suspension point.
- Awaiting in a `Task` creates a suspension point; the UI update may not happen before the next display refresh deadline if it occurs after `await`.
- Pattern: in a `Task`, do pre-work synchronously (set loading state), then `await` the async work, then update state synchronously afterward.

### Structuring Concurrent Code
- **Separate UI logic from async logic**: use a `@State` model as a bridge.
  - UI code reads from the model synchronously; the model drives async work.
  - When async work completes, it mutates model state synchronously, triggering a UI update.
- Keep the code inside `Task` closures minimal — just enough to inform the model of the event.
- This structure also makes async logic independently testable (no SwiftUI import required).
- `Mutex` — use to make a class `Sendable` when actor isolation is not appropriate (see documentation).

## APIs & Frameworks

### SwiftUI
- `View` protocol — `@MainActor`-isolated; source of implicit isolation for conforming types.
- `@State`, `@Binding`, `@Observable` — all `@MainActor`-isolated when used in a `View`.
- `Shape.path(in:)` — `Sendable`; may be called on background thread.
- `visualEffect(_:)` — `Sendable` closure; may be called on background thread.
- `Layout` protocol — methods may be called on background thread.
- `onGeometryChange(for:of:action:)` — transform closure may be called on background thread.
- `onScrollVisibilityChange(threshold:_:)` — synchronous callback; used to trigger animations.
- `withAnimation(_:_:)` — synchronous state mutation that schedules animated frame generation.
- `Task { }` — switches to async context; closure runs on `@MainActor` when invoked from main-actor-isolated code.

### Swift Concurrency
- `@MainActor` — global actor; ensures single-threaded access to UI state.
- `Sendable` — marker protocol/attribute indicating a type or closure is safe to cross actor boundaries.
- `Mutex` — from `Synchronization` framework; makes a class `Sendable` without actor isolation.
- Swift 6.2 implicit `@MainActor` module setting.

## Code Highlights

```swift
// Correct pattern: synchronous loading state + async work
.onTapGesture {
    guard !model.isExtracting else { return }
    withAnimation { model.isExtracting = true }   // synchronous, hits this frame
    Task {
        await model.extractColorScheme()           // async work off-main
        withAnimation { model.isExtracting = false }
    }
}
```

```swift
// Fix: capture value type copy in Sendable closure
.visualEffect { [pulse] content, _ in    // [pulse] captures a Bool copy
    content.blur(radius: pulse ? 2 : 0)
}
```

```swift
// Animate on scroll visibility — synchronous callback is critical
.onScrollVisibilityChange(threshold: 0.9) {
    guard !isShown else { return }
    withAnimation { isShown = $0 }
}
```

## Takeaways
- Trust SwiftUI's `@MainActor` default — you do not need to annotate most code; isolation is inferred from protocol conformance.
- When the compiler flags a `Sendable` violation in a `visualEffect` or `Shape.path` closure, capture a copy of the value you need rather than accessing `self`.
- Keep time-sensitive UI mutations (especially those paired with `withAnimation`) synchronous and before any `await` — suspension can push the update past the display deadline and cause laggy animation.
- Separate your app's async model logic from SwiftUI views; the state-as-bridge pattern makes both layers simpler and independently testable.

---
_Source: WWDC25 Session 266 page (abstract, chapters, full transcript, and code samples)._
