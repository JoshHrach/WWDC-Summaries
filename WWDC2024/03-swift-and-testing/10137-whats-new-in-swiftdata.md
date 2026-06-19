# What's New in SwiftData
**WWDC24 · Session 10137** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10137/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, visionOS 2, watchOS 11, tvOS 18

## Overview
SwiftData's second year adds four major capabilities: `#Unique` compound uniqueness constraints (with automatic upsert on collision), `#Index` for faster predicate queries, `#Expression` for composing arbitrary sub-evaluations inside `#Predicate`, and custom data stores via a new DataStore protocol. Xcode preview integration is also dramatically improved with `PreviewModifier`-based sample data traits and the `@Previewable` macro for inline query declarations. The History API (covered in depth in a companion session) enables tracking model insertions, updates, and deletions over time.

## Key Topics

### `#Unique` — Compound Uniqueness Constraints
`#Unique<ModelType>([\.property1, \.property2, ...])` declares that a combination of properties must remain unique across all model instances. When a new model is inserted with values that match an existing instance, SwiftData performs an **upsert** — merging the new data into the existing model rather than duplicating it. The properties in a `#Unique` constraint also represent the model's identity and can be decorated with `@Attribute(.preserveValueOnDeletion)` to retain their values as tombstone data in the History API when the model is deleted.

### `#Expression` — Complex Sub-Evaluations in Predicates
`#Expression<InputType, OutputType>` is a new Foundation macro that builds reusable sub-evaluations that return arbitrary types (not just `Bool`). Expressions compose with `#Predicate`: you build an expression for a derived value (e.g., count of unplanned bucket list items), then evaluate it inside a predicate to filter models based on that value. This enables complex, efficient query logic that runs in the store rather than in memory.

### `#Index` — Faster Queries
`#Index<ModelType>([\.property], [\.prop1, \.prop2])` adds metadata to the store that functions like a table of contents, making queries that sort or filter on the specified key paths substantially faster. Declare an index for every property or combination of properties you use in frequent predicates or sort descriptors — especially valuable for large data sets.

### Custom Data Stores
A new DataStore protocol allows any persistence backend to work with SwiftData's `@Model` macro and `@Query` property wrapper. Implement the protocol to back SwiftData with JSON files, a remote server, or any custom format. In the session, a `JSONStoreConfiguration` is shown swapping in for `ModelConfiguration` with no changes to model or view code. Custom stores can adopt SwiftData features incrementally (e.g., adding History support later).

### Xcode Preview Integration
**`PreviewModifier` / `PreviewTrait`**: A `SampleData` struct conforming to `PreviewModifier` sets up a shared in-memory `ModelContainer` and pre-populates it with sample models. An extension on `PreviewTrait` exposes it as a static property (`.sampleData`). Any preview can then opt in with `#Preview(traits: .sampleData) { ... }`.

**`@Previewable` macro**: For views that receive models as parameters rather than using `@Query`, `@Previewable @Query var trips: [Trip]` declares a live query directly inside a `#Preview` body — no intermediate wrapper view needed.

### History API
Models decorated with `@Attribute(.preserveValueOnDeletion)` retain their values as tombstone data when deleted. The SwiftData History API exposes a record of all insertions, updates, and deletions over time, enabling sync engines and change-processing workflows. (Covered in "Track model changes with SwiftData history".)

## APIs & Frameworks

**SwiftData**
- `@Model` — no changes; foundational macro (existing)
- `#Unique<T>([KeyPath...])` **[NEW]** — compound uniqueness constraint with upsert on collision
- `#Index<T>([KeyPath...], ...)` **[NEW]** — single and compound index macros for query performance
- `@Attribute(.preserveValueOnDeletion)` **[NEW option]** — retain values as tombstone data in history
- History API **[NEW]** — track model changes over time; see "Track model changes with SwiftData history"
- Custom DataStore protocol **[NEW]** — implement to use any persistence backend
- `ModelConfiguration(schema:url:)` — existing; used to set custom on-disk URL
- `ModelContainer(for:configurations:)` — accepts custom store configurations (existing, expanded)

**Foundation**
- `#Expression<Input, Output>` **[NEW]** — macro for sub-evaluations composable within `#Predicate`
  - `expression.evaluate(_:)` — evaluate expression as part of a predicate

**SwiftUI**
- `@Query` — existing property wrapper, now works with `#Predicate` and `#Expression` compounds
- `PreviewModifier` protocol **[NEW]** — set up shared context for previews (e.g., ModelContainer)
- `PreviewTrait` extension for custom sample data traits **[NEW]**
- `@Previewable` macro **[NEW]** — declare `@Query` or `@State` inline in a `#Preview` body

## Code Highlights

Compound uniqueness constraint with tombstone preservation:
```swift
@Model
class Trip {
    #Unique<Trip>([\.name, \.startDate, \.endDate])

    @Attribute(.preserveValueOnDeletion) var name: String
    var destination: String
    @Attribute(.preserveValueOnDeletion) var startDate: Date
    @Attribute(.preserveValueOnDeletion) var endDate: Date

    var bucketList: [BucketListItem] = []
}
```

`#Expression` inside `#Predicate` for complex filtering:
```swift
let unplannedItemsExpression = #Expression<[BucketListItem], Int> { items in
    items.filter { !$0.isInPlan }.count
}

let today = Date.now
let predicate = #Predicate<Trip> { trip in
    (trip.startDate ..< trip.endDate).contains(today) &&
    unplannedItemsExpression.evaluate(trip.bucketList) > 0
}
```

`#Index` for frequently queried properties:
```swift
@Model
class Trip {
    #Unique<Trip>([\.name, \.startDate, \.endDate])
    #Index<Trip>([\.name], [\.startDate], [\.endDate], [\.name, \.startDate, \.endDate])

    var name: String
    var destination: String
    var startDate: Date
    var endDate: Date
}
```

Preview with `PreviewModifier` sample data trait:
```swift
struct SampleData: PreviewModifier {
    static func makeSharedContext() throws -> ModelContainer {
        let config = ModelConfiguration(isStoredInMemoryOnly: true)
        let container = try ModelContainer(for: Trip.self, configurations: config)
        Trip.makeSampleTrips(in: container)
        return container
    }
    func body(content: Content, context: ModelContainer) -> some View {
        content.modelContainer(context)
    }
}
extension PreviewTrait where T == Preview.ViewTraits {
    @MainActor static var sampleData: Self = .modifier(SampleData())
}

#Preview(traits: .sampleData) {
    ContentView()
}
```

`@Previewable` for views receiving model parameters:
```swift
#Preview(traits: .sampleData) {
    @Previewable @Query var trips: [Trip]
    BucketListItemView(trip: trips.first)
}
```

Custom data store usage:
```swift
let configuration = JSONStoreConfiguration(schema: Schema([Trip.self]), url: jsonFileURL)
let container = try ModelContainer(for: Trip.self, configurations: configuration)
```

## Takeaways
- Add `#Unique` to every model that can have duplicates — SwiftData will automatically upsert on collision, eliminating manual deduplication code.
- Use `#Expression` to express complex derived values (counts, sums) in queries rather than loading and filtering large datasets in memory.
- Apply `#Index` to the properties and compound combinations you most frequently sort or filter on; for large data sets, this makes a measurable difference in query speed.
- The `PreviewModifier` + `@Previewable` combination gives every view in the app realistic, full-fidelity previews without additional boilerplate.

---
_Source: WWDC24 Session 10137 page (abstract, chapter summaries, code samples, and resource links)._
