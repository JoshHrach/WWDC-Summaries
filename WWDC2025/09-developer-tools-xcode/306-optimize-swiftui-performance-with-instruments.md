# Optimize SwiftUI performance with Instruments
**WWDC25 · Session 306** · [Watch](https://developer.apple.com/videos/play/wwdc2025/306/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26

## Overview
A new SwiftUI instrument arrives in Instruments, providing direct visibility into how SwiftUI evaluates view bodies, propagates state changes, and performs diffing. Previously, diagnosing SwiftUI performance required guesswork about which views were re-evaluating unnecessarily. The new instrument makes this explicit: every view body call appears as a named, timestamped event in the timeline, annotated with the state change that triggered it.

The session uses a realistic document-management app to demonstrate the complete debugging workflow: recording a trace, identifying unexpectedly frequent view body calls, tracing them back to a specific `@State` or `@ObservedObject` property, and applying targeted fixes. It covers how changes in observable data ripple through the SwiftUI view tree and how to break unnecessary dependency chains.

## Key Topics

### The SwiftUI Instrument
The new SwiftUI instrument adds a dedicated track to Instruments that shows view body evaluation events. Each event displays the view type name, the duration of the body call, and — critically — which property change triggered it. This eliminates the need to add print statements or SwiftUI's debug modifiers to understand update causality.

### How SwiftUI Updates Views
SwiftUI re-evaluates a view's body whenever any property that the body reads changes. The instrument visualizes this dependency graph at runtime. The session explains the three sources of updates: `@State` changes, `@ObservedObject`/`@Observable` property changes, and environment value changes.

### Data Flow and Dependency Chains
A common performance anti-pattern is placing a frequently-updating value (e.g., a timer or a location coordinate) at a high level in the view tree, causing the entire subtree to re-evaluate on every tick. The instrument makes this visible as a storm of view body events cascading from a single property.

The recommended fix is to push the frequently-updating state as close to the consuming view as possible, or to use `@Observable` with selective observation to narrow which properties each view depends on.

### Identifying and Fixing Unnecessary Updates
The session walks through:
1. Spotting views with unexpectedly high body call counts in the summary view.
2. Inspecting the triggering property for each call.
3. Refactoring data models to isolate volatile state.
4. Verifying the fix reduced body calls without breaking correctness.

### `@Observable` vs. `ObservableObject`
The session compares the two observation systems. `@Observable` (Swift Observation macro) provides per-property tracking — a view only re-evaluates when the specific properties it reads change. `ObservableObject` with `@Published` invalidates the entire view on any `objectWillChange` emission. For performance-sensitive views, migrating to `@Observable` is frequently the highest-leverage change.

## APIs & Frameworks

- **SwiftUI Instrument** **[NEW in Instruments]** — view body evaluation timeline with triggering-property annotation
- **`@Observable` macro** (Swift Observation framework) — per-property dependency tracking for minimal view updates
- **`@ObservedObject` / `ObservableObject`** (existing) — whole-object invalidation model
- **`@State`** (existing) — local view state, body re-evaluation scoped to owning view
- **`@Environment`** (existing) — environment value dependencies shown in instrument
- **`withAnimation`** (existing) — animation transactions, visualized as correlated events

## Code Highlights

```swift
// Anti-pattern: high-frequency update at root causes full subtree re-evaluation
class LocationModel: ObservableObject {
    @Published var coordinate: CLLocationCoordinate2D  // updates every second
    @Published var destinationName: String             // updates rarely
}

struct RootView: View {
    @ObservedObject var model: LocationModel  // any publish invalidates all children
    var body: some View {
        VStack {
            MapView(coordinate: model.coordinate)
            DestinationLabel(name: model.destinationName)  // re-evaluated every second unnecessarily
        }
    }
}
```

```swift
// Fix: use @Observable for per-property tracking
@Observable
class LocationModel {
    var coordinate: CLLocationCoordinate2D  // only views reading this re-evaluate
    var destinationName: String             // only views reading this re-evaluate
}

struct DestinationLabel: View {
    var model: LocationModel
    var body: some View {
        Text(model.destinationName)  // only re-evaluates when destinationName changes
    }
}
```

## Takeaways

- The SwiftUI instrument is the definitive tool for view body performance — use it before reaching for manual print debugging or `_printChanges()`.
- Per-property observation with `@Observable` is almost always more efficient than `ObservableObject` for models with a mix of fast-changing and slow-changing properties.
- Push frequently-updating state down the view tree as close to the consumer as possible; every level it descends reduces the number of views that re-evaluate.
- High body-call counts on leaf views are often symptoms of a data model problem, not a view problem.

---
_Source: WWDC25 Session 306 page (abstract, chapter summaries, code samples, and resource links)._
