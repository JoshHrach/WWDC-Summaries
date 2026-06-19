# Demystify SwiftUI Performance
**WWDC23 · Session 10160** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10160/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, watchOS 10, tvOS 17, visionOS 1

## Overview
This session builds a mental model for SwiftUI performance so developers can write fast code from the start rather than debugging regressions later. It covers three interconnected areas: understanding the SwiftUI dependency graph and how to reduce unnecessary updates; identifying and eliminating slow work inside `body` and during dynamic property initialization; and understanding how `List` and `Table` gather identifiers eagerly and how to keep that process efficient.

Prerequisites: familiarity with SwiftUI identity (implicit vs. explicit), view lifetime, and the view value graph from "Demystify SwiftUI" (WWDC21).

## Key Topics

### The Dependency Graph and Update Process
SwiftUI maintains a graph where each node is a view and edges represent parent-child relationships through `body`. When data changes, SwiftUI walks the graph and re-evaluates only views whose dependencies have changed.

A view's dependencies come from two sources:
1. **The view value itself** — all stored properties passed from the parent.
2. **Dynamic properties** — `@State`, `@Environment`, `@Binding`, `@Observable`, etc.

When a dependency changes, SwiftUI produces a new view value, updates all dynamic properties, then calls `body`. This recurses down the graph.

### Diagnosing Unnecessary Updates: `_printChanges()`
The debugging tool `Self._printChanges()` (called from `body` or LLDB) prints why SwiftUI called into a view's `body`:
- Shows which stored property changed (`@Self changed`) or which dynamic property changed.
- Debugging-only: never submit calls to `_printChanges()` to the App Store; remove them after investigation.
- Use from LLDB at a breakpoint: `expression Self._printChanges()`

**Pattern**: If `_printChanges` reports `@Self changed` when only an unrelated property of a large model object changed, the view has an over-broad dependency.

### Reducing Dependencies
Fix over-broad dependencies by narrowing the view value:
- Pass only the specific data the view needs (e.g., `Image` instead of the full `Dog` struct to `ScalableDogImage`).
- Extract subviews: give each extracted view only the properties it needs. Dependencies are then apparent at the use site, and unrelated changes no longer trigger that subview's `body`.
- Use `@Observable` (new in iOS 17 / macOS Sonoma): automatically tracks only the specific properties read during `body`, eliminating false dependencies at the object level. See "Discover Observation in SwiftUI" (10149).

**Caution**: narrowing is most valuable for small, frequently-updated subviews. Very large structs may not benefit if most properties are always read.

### Faster Body Execution
Common causes of slow `body` execution:
- **Expensive `@State` / `@Observable` initialization**: move slow work (network requests, disk I/O) out of `init()` and into `.task { }` or other async contexts.
- **Inline data filtering**: filtering or sorting inside `body` runs on every update. Cache filtered collections in the model instead.
- **String interpolation**: can allocate; cache formatted strings.
- **Bundle lookups**: repeated `Bundle.main.url(forResource:)` calls can be costly; cache results.
- **Heap allocations**: class instantiation in body adds up; prefer value types or pre-create objects.

Example fix — move fetch out of `init` into `.task`:
```swift
// Before: fetchDogs() blocks init, runs synchronously
@Observable class FetchModel {
    var dogs: [Dog]
    init() { fetchDogs() } // slow
}

// After: async fetch in .task modifier
struct DogRootView: View {
    @State private var model = FetchModel()
    var body: some View {
        DogList(model.dogs)
            .task { await model.fetchDogs() }
    }
}
@Observable class FetchModel {
    var dogs: [Dog] = []
    func fetchDogs() async { /* ... */ }
}
```

### Identity in List and Table
`List` and `Table` gather **all row identifiers eagerly** before rendering any content — this is what enables incremental updates, animations, and reordering. Identification performance directly drives load and update speed.

**The key rule**: within a `ForEach` inside `List` or `Table`, the number of views produced per data element must be **constant**.

Why it matters: if the view count per element varies, `List` cannot compute row IDs without building all the views first — defeating lazy loading. If the count is constant, `List` only builds the views for visible rows plus a prefetch buffer.

**Anti-patterns**:
```swift
// BAD: variable view count (0 or 1) per element forces building all rows
ForEach(dogs) { dog in
    if dog.favoritesTennisBall { DogCell(dog) }  // conditional = variable count
}

// BAD: AnyView — unknown row count, forces building all rows
ForEach(dogs) { dog in
    AnyView(DogCell(dog))
}
```

**Fix**: pre-filter in the model; each element always produces exactly 1 view:
```swift
// GOOD: constant 1 view per element; filtering moved to model
ForEach(tennisBallDogs) { dog in
    DogCell(dog)
}
```

**Sections with nested ForEach**: dynamic sections using `ForEach` → `Section` → `ForEach` is a well-understood pattern; `List` handles this correctly and efficiently.

**Table improvements (iOS 17 / macOS Sonoma)**:
- New streamlined `ForEach` initializer in `Table` rows closure: `ForEach(dogs)` — no explicit `TableRow` wrapper needed. Back-deploys to all OS versions where `Table` is available.
- **Semantic change**: In iOS 17+, row identity is derived from the `ForEach` element's `id`, not the `TableRow`'s value. If you pass a different value to `TableRow` than the `ForEach` element, the IDs now come from the `ForEach` element. Fix by mapping or using explicit `id:` key path.

**General performance improvements in iOS 17 / macOS Sonoma**: SwiftUI has under-the-hood optimizations for list/table filtering and scrolling with no code changes required.

## APIs & Frameworks

**SwiftUI**
- `Self._printChanges()` — debug-only; prints dependency change reason from within `body`
- `expression Self._printChanges()` — LLDB command form
- `View.task(_:)` — run async work when view appears; safe for slow initialization
- `@Observable` (Observation framework, iOS 17) — fine-grained property-level dependency tracking
- `ForEach` in `List`/`Table` — constant view count per element for efficient ID gathering
- `Table` `rows:` `ForEach(collection)` shorthand **[NEW iOS 17]** — implicit `TableRow` wrapping, back-deploys

## Code Highlights

Narrowing view dependencies:
```swift
// Before: ScalableDogImage depends on entire Dog struct
struct ScalableDogImage: View {
    @State private var scaleToFill = false
    var dog: Dog  // over-broad: any Dog property change triggers body
    var body: some View {
        dog.image.resizable()...
    }
}

// After: only depends on the Image — unrelated Dog changes ignored
struct ScalableDogImage: View {
    @State private var scaleToFill = false
    var image: Image  // only what's needed
    var body: some View {
        image.resizable()...
    }
}
```

Using `_printChanges` to investigate:
```swift
var body: some View {
    let _ = Self._printChanges()  // REMOVE BEFORE SHIPPING
    // ... rest of body
}
```

Efficient sectioned list (nested `ForEach` — recommended pattern):
```swift
List {
    ForEach(model.dogToys) { toy in
        Section(toy.name) {
            ForEach(model.dogs(toy: toy)) { dog in
                DogCell(dog)
            }
        }
    }
}
```

New Table ForEach shorthand (iOS 17):
```swift
Table(of: Dog.self) {
    // columns...
} rows: {
    ForEach(dogs)  // implicitly wraps each element in TableRow
}
```

## Resources
- Related: "Demystify SwiftUI" (WWDC21 10022) — prerequisite on identity and lifetime
- Related: "Discover Observation in SwiftUI" (WWDC23 10149) — `@Observable` for dependency scoping
- Related: "Analyze hangs with Instruments" (WWDC23 10248) — profiling SwiftUI hangs
- Related: "Explore UI animation hitches and the render loop" (Tech Talks 10855)

## Takeaways
- Use `Self._printChanges()` during debugging to identify which dependency triggered a `body` call; look for `@Self changed` on views that shouldn't care about the changed data.
- Narrow view values to only the data each view actually uses; extract subviews to make dependencies explicit and minimize update scope.
- Move all slow work (I/O, network, filtering) out of `body` and `init` into `.task` or cached model properties — `body` must be fast.
- In `List` and `Table`, every `ForEach` content closure must produce a constant number of views per element; pre-filter data in the model rather than using in-body conditionals or `AnyView`.

---
_Source: WWDC23 Session 10160 page (abstract, transcript, chapter summaries, code samples, and resource links)._
