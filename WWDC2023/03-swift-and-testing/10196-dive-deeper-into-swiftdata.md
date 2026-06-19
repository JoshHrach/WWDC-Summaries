# Dive Deeper into SwiftData
**WWDC23 · Session 10196** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10196/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1

## Overview
Nick Gillett from the SwiftData team dives into the internals of the framework, covering three areas that go beyond the basics in "Meet SwiftData": configuring the ModelContainer with complex persistence requirements, understanding how the ModelContext tracks changes in memory and persists them, and writing performant large-scale queries with `FetchDescriptor`, `SortDescriptor`, and `enumerate`. This session uses the SampleTrips app to illustrate real patterns like multi-store configurations, undo/redo, autosave, and batch enumeration.

## Key Topics

### The Model Duality: Schema + Instance Interface
A class annotated with `@Model` plays two roles simultaneously:
1. **Schema role**: describes the object graph for the `ModelContainer` (which generates the database structure).
2. **Instance role**: acts as a Swift type developers write code against.

These two roles are served by two aligned abstractions: `ModelContainer` (for persistence) and `ModelContext` (for in-memory tracking).

### Configuring Persistence with ModelContainer and ModelConfiguration **[NEW]**
`ModelContainer` bridges the schema to its backing store. It can be configured at different levels of complexity:
- **Simple**: `try ModelContainer(for: Trip.self)` — SwiftData infers related model types automatically.
- **Advanced**: `ModelConfiguration` describes one schema segment's persistence: file URL, in-memory vs. on-disk, read-only mode, and CloudKit container identifier.

Multiple `ModelConfiguration` objects can be combined into a single `ModelContainer` to support separate object graphs (e.g., trips data and contacts data stored in different files with different CloudKit containers). The full schema must include all types across all configurations.

In SwiftUI apps, the `.modelContainer(for:)` scene modifier is the preferred way to create and inject the container.

### ModelContext: Change Tracking and Persistence **[NEW]**
The `ModelContext` is the in-memory view over managed data:
- Fetched objects are held in the context until explicitly freed.
- Edits are tracked as snapshots; deletes remove objects from UI but they remain in the context until `save()` is called.
- `context.save()` persists all pending changes to the container and clears tracked state.
- `context.rollback()` / `context.reset()` discard cached changes.

The `modelContainer` SwiftUI modifier binds the environment's `\.modelContext` key to the container's **mainContext** — a MainActor-aligned context for use in scenes and views.

**Undo/Redo**: The `modelContainer(for:isUndoEnabled:)` modifier binds the window's `undoManager` to the mainContext. Changes are automatically registered as undo actions; three-finger swipe and device shake trigger undo/redo with zero additional code.

**Autosave**: Enabled by default. The mainContext saves automatically on app foreground/background transitions and periodically during use. Disable with `modelContainer(for:isAutosaveEnabled:false)`. Hand-created `ModelContext` objects have autosave disabled by default.

### Querying at Scale: FetchDescriptor, Predicate, and enumerate **[NEW]**
`FetchDescriptor<T>` is a strongly-typed fetch request using generics — no casting needed, compiler validates property references:
- Combines with the `#Predicate` macro for compiler-validated queries, including subqueries and joins written in pure Swift.
- `SortDescriptor(\T.property)` provides type-safe sort ordering.
- Tuning options: `offset`, `limit`, faulting, and prefetching.

`ModelContext.enumerate(_:batchSize:allowEscapingMutations:)` is the correct pattern for large batch traversals:
- Default `batchSize` is 5,000; reduce for heavy object graphs (images, video), increase to reduce I/O for lightweight ones.
- Default behavior includes a **mutation guard**: if the context is dirty (has pending changes) when enumerate is called, it throws to prevent objects from being trapped and preventing deallocation.
- Set `allowEscapingMutations: true` to intentionally allow mutations during enumeration.

## APIs & Frameworks

### SwiftData **[NEW]**
- `@Model` macro — marks a class as a SwiftData model; synthesizes schema and persistence hooks **[NEW]**
- `@Relationship(_:)` — customizes relationship behavior (e.g., `.cascade` delete rule) **[NEW]**
- `ModelContainer` — bridges schema to persistence backend **[NEW]**
  - `ModelContainer(for:)` — convenience initializer inferring related types **[NEW]**
  - `ModelContainer(for:configurations:)` — multi-configuration initializer **[NEW]**
- `ModelConfiguration` — describes one schema segment's persistence **[NEW]**
  - `init(schema:url:isStoredInMemoryOnly:readOnly:cloudKitContainerIdentifier:)` **[NEW]**
- `ModelContext` — in-memory change tracker **[NEW]**
  - `modelContext` environment key (`\.modelContext`) — bound by `.modelContainer` modifier **[NEW]**
  - `context.save()` — persist all pending changes **[NEW]**
  - `context.rollback()` — discard all pending changes **[NEW]**
  - `context.reset()` — clear all cached objects **[NEW]**
  - `context.insert(_:)` — insert a new model object **[NEW]**
  - `context.delete(_:)` — mark an object for deletion **[NEW]**
  - `context.fetch(_:)` — fetch objects matching a descriptor **[NEW]**
  - `context.enumerate(_:batchSize:allowEscapingMutations:)` — batch traversal **[NEW]**
- `FetchDescriptor<T>` — type-safe fetch request **[NEW]**
  - `FetchDescriptor(predicate:sortBy:)` **[NEW]**
  - `.offset`, `.limit`, `.fetchLimit`, `.fetchOffset` — pagination parameters **[NEW]**
- `SortDescriptor<T>(\T.property)` — type-safe sort key **[NEW]**
- `#Predicate<T>` macro — compiler-validated predicate supporting subqueries and joins **[NEW]**
- `@Query` property wrapper — SwiftUI property for live-updating fetch results **[NEW]**
- `Schema` — collection of model types describing the full object graph **[NEW]**

### SwiftUI Integration **[NEW]**
- `.modelContainer(for:)` — scene/view modifier; creates container and binds `\.modelContext` **[NEW]**
- `.modelContainer(for:isUndoEnabled:)` — enables undo/redo via window's `undoManager` **[NEW]**
- `.modelContainer(for:isAutosaveEnabled:)` — disables autosave if needed **[NEW]**
- `@Environment(\.modelContext)` — access the main context in a view **[NEW]**

## Code Highlights

```swift
// @Model with cascading relationship
@Model final class Trip {
    var name: String?
    var destination: String?
    var start_date: Date?
    var end_date: Date?
    @Relationship(.cascade) var bucketListItem: [BucketListItem] = []
    @Relationship(.cascade) var livingAccommodation: LivingAccommodation?
}
```

```swift
// Multi-store ModelContainer
let fullSchema = Schema([Trip.self, BucketListItem.self,
                         LivingAccommodations.self, Person.self, Address.self])
let trips = ModelConfiguration(
    schema: Schema([Trip.self, BucketListItem.self, LivingAccommodations.self]),
    url: URL(filePath: "/path/to/trip.store"),
    cloudKitContainerIdentifier: "com.example.trips"
)
let people = ModelConfiguration(
    schema: Schema([Person.self, Address.self]),
    url: URL(filePath: "/path/to/people.store"),
    cloudKitContainerIdentifier: "com.example.people"
)
let container = try ModelContainer(for: fullSchema, trips, people)
```

```swift
// SwiftUI app with undo and autosave disabled
@main struct TripsApp: App {
    var body: some Scene {
        WindowGroup { ContentView() }
            .modelContainer(for: Trip.self, isUndoEnabled: true, isAutosaveEnabled: false)
    }
}
```

```swift
// FetchDescriptor with #Predicate and SortDescriptor
let predicate = #Predicate<Trip> { trip in
    trip.livingAccommodations.filter { $0.hasReservation == false }.count > 0
}
let descriptor = FetchDescriptor(predicate: predicate,
                                 sortBy: [SortDescriptor(\.start_date)])
let trips = try context.fetch(descriptor)
```

```swift
// enumerate for batch processing
context.enumerate(descriptor, batchSize: 500, allowEscapingMutations: true) { trip in
    // Process each trip; allowEscapingMutations: true permits in-loop mutations
}
```

## Takeaways
- Use `ModelConfiguration` to define separate persistence stores for different object graphs in the same container — critical for apps with mixed CloudKit container requirements.
- Always hold `ModelContext` objects strongly (as properties, not local variables); deallocation ends their tracking scope. The mainContext from the environment is the right context for all SwiftUI view work.
- Enable `isUndoEnabled: true` on the model container to get system undo/redo gestures for free — no `UndoManager` code needed.
- Use `context.enumerate(_:batchSize:allowEscapingMutations:)` instead of `fetch` for large batch traversals; it enforces platform best practices for memory and mutation safety automatically.

---
_Source: WWDC23 Session 10196 page (abstract, chapter summaries, code samples, and resource links)._
