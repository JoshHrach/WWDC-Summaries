# What's New in SwiftData
**WWDC26 · Session 274** · [Watch](https://developer.apple.com/videos/play/wwdc2026/274/)

_Platforms:_ iOS, iPadOS, macOS, tvOS, visionOS, watchOS

## Overview
This session covers four targeted enhancements to SwiftData that expand its usefulness in real-world app architectures. The additions address common gaps: grouping fetched results into sections for SwiftUI lists, persisting types that SwiftData cannot natively model (such as third-party SDK types), observing model changes from non-SwiftUI code, and reacting to fine-grained persistent-history transactions for syncing or auditing purposes.

The session demonstrates all features using the SampleTrips app and a server-sync scenario, showing how each feature integrates with the broader SwiftUI and Swift Observation frameworks. The new observer types — `ModelResultsObserver` and `HistoryObserver` — bring `@Query`-style reactivity to view models, actors, and any other code that cannot use property wrappers directly.

## Key Topics

### Sectioned Fetching with `@Query(sectionBy:)`
A new `sectionBy:` parameter on `@Query` groups fetched results by a key path. The query result type remains a flat array, but the underlying query object (accessed via the underscore-prefix, e.g., `_trips`) exposes a `sections` property. Each section has an `id` (the grouped value) and a collection of models. Use `ForEach(_trips.sections)` to render section headers in a SwiftUI `List`, with `ForEach(section.models)` for the rows. Both `sort:` and `sectionBy:` can be specified together.

### Persisting Custom and Third-Party Types with `@Attribute(.codable)`
SwiftData natively stores a set of supported value types. When you need to store a type you do not own — such as `MKMapItem.Identifier` from MapKit — mark the property with `@Attribute(.codable)`. SwiftData will serialize it using the type's `Codable` conformance. The session emphasizes this is an escape hatch: for types you own, prefer modeling them as `@Model` classes or supported value types to retain filtering, sorting, predicate support, and schema migration capabilities. A custom struct (e.g., `Location` with `latitude`/`longitude`) should simply conform to `Codable` and be stored without the attribute; the attribute is reserved for types where you cannot add `@Model`.

### Observing Data Stores: `ModelResultsObserver`
The new `ModelResultsObserver<ModelType>` type brings `@Query`-equivalent fetching to non-SwiftUI contexts. Initialize it with a `ModelContext` and optionally a `FetchDescriptor`. Its `results` property is observable via Swift Observation. Use `withContinuousObservation(options: [.didSet]) { ... }` to subscribe to changes and run arbitrary code (update a map camera bounds, drive a view model property, etc.) whenever the underlying data changes. This is the correct tool for `@Observable` view models, coordinator objects, and any non-view code that needs to react to model changes.

### Observing History: `HistoryObserver`
The new `HistoryObserver` wraps persistent history tracking in a single observable object. Initialize it with an `authors` array (strings matching the `ModelContext.author` used when saving changes) and a `ModelContainer`. Its `eventCounter` property increments each time new history transactions arrive for the specified authors. Observe `eventCounter` with `withContinuousObservation(options: .didSet)` and respond by calling `ModelContext.fetchHistory()` to retrieve the actual transactions. This pattern is ideal for server-sync actors, background workers, and audit logging — scenarios where you need to know exactly which model instances changed and how, not just that the store changed.

## APIs & Frameworks

**SwiftData — new/changed**
- **[NEW]** `@Query(sort:sectionBy:)` — sectioned fetch with key-path grouping
- **[NEW]** `SectionedQueryResults.sections` — accessed via underscore-prefixed query variable (e.g., `_trips.sections`)
- **[NEW]** `QuerySection` — has `id` (the section grouping value) and a collection of models
- **[NEW]** `@Attribute(.codable)` — persists `Codable`-conforming types that SwiftData cannot natively model
- **[NEW]** `ModelResultsObserver<T: PersistentModel>` — `@Query`-equivalent observer for non-SwiftUI code
  - `init(modelContext:)` / `init(modelContext:fetchDescriptor:)`
  - `results: [T]` — observable property
- **[NEW]** `HistoryObserver` — persistent-history change observer
  - `init(authors:[String], modelContainer:)`
  - `eventCounter: Int` — observable, increments on new transactions
- `ModelContext.fetchHistory()` — fetch persistent-history transactions (existing, used with `HistoryObserver`)
- `ModelContext.author` — string tag applied to saves, used to filter history

**SwiftData — existing (referenced)**
- `@Model` macro
- `@Attribute` macro (various options)
- `@Relationship(deleteRule:inverse:)` macro
- `@Query(filter:sort:)` / `@Query(sort:)` property wrapper
- `FetchDescriptor<T>` / `#Predicate`
- `ModelContainer(for:)` / `ModelContext`
- `PersistentModel` protocol

**Swift Observation (used with new APIs)**
- `withContinuousObservation(options:_:)` — subscribe to observable property changes
- `ObservationTracking.Token` — cancellation token for observation
- `ObservationTracking.Event` — event passed to observation closure

**MapKit (integration example)**
- `MKMapItem.Identifier` — third-party Codable type stored via `@Attribute(.codable)`
- `MapCameraBounds` — derived from observed `ModelResultsObserver.results`

## Code Highlights

**Sectioned `@Query`:**
```swift
@Query(sort: \Trip.startDate, sectionBy: \.destination)
var trips: [Trip]

// In body:
ForEach(_trips.sections) { section in
    Section(section.id) {
        ForEach(section.models) { trip in TripListItem(trip: trip) }
    }
}
```

**`@Attribute(.codable)` for third-party type:**
```swift
@Model class Trip {
    var location: Location?  // own Codable struct — no attribute needed
    @Attribute(.codable) var mapItemIdentifier: MKMapItem.Identifier?  // third-party type
}
```

**`ModelResultsObserver` in an `@Observable` view model:**
```swift
@Observable @MainActor final class MapCameraController {
    private let modelResultsObserver: ModelResultsObserver<Trip>
    var bounds: MapCameraBounds?

    init(modelContext: ModelContext) throws {
        modelResultsObserver = try ModelResultsObserver<Trip>(modelContext: modelContext)
        token = withContinuousObservation(options: [.didSet]) { [weak self] _ in
            self?.bounds = self?.calculateBounds(trips: self!.modelResultsObserver.results)
        }
    }
}
```

**`HistoryObserver` for server sync:**
```swift
@SyncActor final class ServerSync {
    func start() throws {
        observer = try HistoryObserver(authors: ["App"], modelContainer: modelContainer)
        token = withContinuousObservation(options: .didSet) { [weak self] _ in
            _ = self?.observer.eventCounter
            self?.processChanges()
        }
    }
}
```

## Takeaways
- Use `@Query(sectionBy:)` to replace manual grouping logic in `List` views — sections are driven directly from the store with no extra state.
- Use `@Attribute(.codable)` as a precise escape hatch for third-party types; avoid using it for types you own, as it sacrifices predicate filtering and migration support.
- Replace custom change-notification schemes with `ModelResultsObserver` in view models and actors — it gives the same reactive behavior as `@Query` without requiring SwiftUI property wrappers.
- Adopt `HistoryObserver` + `ModelContext.fetchHistory()` for any sync or audit scenario that needs to know the exact set of model changes, not just that something changed.

---
_Source: WWDC26 Session 274 page (abstract, chapter summaries, code samples, and resource links)._
