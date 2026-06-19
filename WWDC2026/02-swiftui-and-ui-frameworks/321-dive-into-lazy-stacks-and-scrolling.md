# Dive into Lazy Stacks and Scrolling with SwiftUI
**WWDC26 · Session 321** · [Watch](https://developer.apple.com/videos/play/wwdc2026/321/)

_Platforms:_ iOS, iPadOS, macOS, watchOS, visionOS, tvOS

## Overview
This deep-dive session explains exactly how `LazyVStack` and `LazyHStack` work under the hood — how they estimate content sizes, load and unload subviews, prefetch off-screen content, and coordinate with a parent `ScrollView`. Understanding these mechanics is essential for avoiding the common performance pitfalls that produce jank in long scrolling lists.

The session is presented as an evolving origami-instructions app. Starting with a basic `LazyVStack` of steps, it progressively adds a nested horizontal `LazyHStack` showcase, pinned section headers, scroll transitions, and programmatic scrolling — while at each step explaining the underlying layout and lifecycle rules that determine whether the implementation will be fast or slow.

The key insight threading the whole session: lazy stacks work with estimates, not exact sizes, and anything that forces a re-layout after a subview appears (size changes from `onGeometryChange`, filtering via conditional view content) degrades scroll quality. The cure is almost always to move logic into the initializer or data layer, not into `onAppear`.

## Key Topics

### Layout (1:24)
`LazyVStack` measures the available width and asks each subview for its ideal height. Subviews that haven't yet appeared use an estimated height derived from previously measured subviews. As the user scrolls, those estimates are refined and the scroll offset is adjusted correspondingly. Composed layouts (a vertical stack containing a horizontal stack) work the same way — the outer stack just treats the inner scroll view as a single, fixed-height subview.

Pinned headers (`LazyVStack(pinnedViews: [.sectionHeaders])`) stay in the viewport while their section content scrolls; scroll transitions (`scrollTransition`) apply per-item visual effects keyed to scroll phase.

`onScrollGeometryChange` reads the absolute content offset, while `onScrollTargetVisibilityChange` fires when a given percentage of a specific item enters the viewport — the right tool for "show scroll-to button once the user has passed the intro section."

### Subview Loading (9:13)
A single `ForEach` item can resolve to zero, one, or multiple subviews. When a `StepView` body conditionally returns nothing (e.g., `if step.isVisible(in: detailLevel)`), the lazy stack sees a gap and lays out accordingly — this is expensive if the condition changes after layout. Always filter at the data level (`@Query` with `#Predicate`) or in the initializer, not inside the view's body.

### Prefetching (13:15)
Lazy stacks prefetch a few subviews ahead of the visible region to do partial layout work before they appear. Taking advantage of this requires initializing view state in `init` (or as a `State` initial value) rather than in `onAppear`, because `onAppear` fires *after* the subview appears on screen — too late to hide the hitch. Use `.task` for async work triggered by the subview being ready.

State that must outlive scrolling (highlight sets, selection) belongs in an outer `@State` binding or model object, not in the subview's own `@State`, because the lazy stack may evict and re-create the subview.

### Programmatic Scrolling (17:40)
`ScrollPosition` + `.scrollPosition($scrollPosition)` scrolls to a target by ID even when the target is off-screen. The lazy stack estimates the destination's position and refines as it approaches. All the same pitfalls apply: dynamic subview counts and layout passes from `onAppear`/`onGeometryChange` make the estimated scroll position jump. A custom `Layout` conformance can eliminate those layout passes entirely.

## APIs & Frameworks

**SwiftUI**
- `LazyVStack(pinnedViews:)` — pinned section headers/footers
- `LazyHStack` — horizontal lazy layout
- `ScrollView` — vertical / horizontal / both
- `.scrollTransition { effect, phase in }` modifier — per-item scroll phase effects
- `ScrollTransitionPhase.value` — -1...1 continuous phase value
- `onScrollGeometryChange(for:of:action:)` — observe absolute offset / content size
- `onScrollTargetVisibilityChange(idType:threshold:)` — observe item visibility fraction
- `ScrollPosition` — programmatic scroll target
- `.scrollPosition(_:)` modifier
- `ScrollViewReader` + `scrollProxy.scrollTo(_:anchor:)` (existing)
- `ForEach` — multiple or conditional subview resolution
- `@Query` with `#Predicate` — data-level filtering (SwiftData)
- `@State` macro — lazy initialization; init assignment pattern
- `.task { }` modifier — async work tied to view lifetime
- `Section` with `header:` — used inside lazy stacks for pinning

**Layout**
- `Layout` protocol — custom layout to avoid `onGeometryChange` re-layout issues

## Code Highlights

Scroll transition on horizontal showcase items:
```swift
PhotoView(photo: photo)
    .scrollTransition { effect, phase in
        effect.scaleEffect(1 - abs(phase.value) * 0.1)
    }
```

Data-level filtering to avoid conditional subviews:
```swift
@Query var steps: [Step]
init(detailLevel: DetailLevel) {
    _steps = Query(filter: #Predicate<Step> { $0.detailLevel >= detailLevel })
}
```

Initialize state in `init` instead of `onAppear`:
```swift
struct StepView: View {
    @State var diagramLoader: DiagramLoader
    init(step: Step) {
        self.step = step
        _diagramLoader = State(initialValue: DiagramLoader(id: step.id))
    }
}
```

Programmatic scroll to a named section header:
```swift
scrollPosition.scrollTo(id: "showcase-header")
```

## Takeaways
- Never use `UIScrollView.contentSize` / absolute offsets for logic in lazy stacks — the content size is always an estimate and changes as views are measured.
- Filter data at the model/query level, not with conditional view content inside subview bodies; each conditional that changes count forces re-estimation.
- Set up view models and loaders in `init` (or as `@State` initial values), not in `onAppear`, to give prefetching a chance to do its work before views appear.
- Keep cross-subview state (selection, highlights) in a binding from the parent or in a model object — the lazy stack will evict subviews and local `@State` is lost.

---
_Source: WWDC26 Session 321 page (abstract, chapter summaries, code samples, and resource links)._
