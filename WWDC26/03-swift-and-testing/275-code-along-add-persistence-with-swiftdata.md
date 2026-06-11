# Code-along: Add Persistence with SwiftData
**WWDC26 · Session 275** · [Watch](https://developer.apple.com/videos/play/wwdc2026/275/)

_Platforms:_ iOS, iPadOS, macOS, tvOS, visionOS, watchOS

## Overview
This hands-on code-along session adds SwiftData persistence to "Wishlist," an existing SwiftUI travel-planning app. The session is structured around three foundational steps that apply to any SwiftData adoption: identifying relevant state, defining schemas, and defining model relationships. Working through a real, non-trivial app — with multiple model types, a class hierarchy, computed properties with observers, and dynamic queries — makes it an ideal reference for production adoption.

The session covers several non-obvious modeling decisions: how to handle Swift `enum` types that need richer behavior using class inheritance, how `@Model` interacts with property observers, how to re-enable property-change side effects after adoption using the new `withContinuousObservation` API, and how to surface SwiftData runtime errors to users via SwiftUI error-handling modifiers.

By the end, the app's ephemeral in-memory `DataSource` and `TripEditModel` helpers are completely replaced by `ModelContainer`, `ModelContext`, and `@Query`-driven views.

## Key Topics

### Step 1: Identify Relevant State
Map existing app state to the SwiftData model layer. The session identifies `Trip`, `Activity`, `TripImage`, and `Goal` (and its subclasses `TripGoal`, `ActivityGoal`) as persistent models, and the singleton `DataSource` as the container to be replaced by `ModelContainer` + `ModelContext`.

### Step 2: Define Your Schemas

**Converting to `@Model`:** Add `@Model` to each class. The macro automatically generates `Observable` conformance, so `@Observable` can be removed. Property observers (`willSet`/`didSet`) are not invoked by SwiftData's property access patterns — side effects must be re-implemented using `withContinuousObservation`.

**Custom value types:** A custom `struct Location: Codable` can be stored directly as a property. A third-party type like `MKMapItem.Identifier` requires `@Attribute(.codable)`.

**Modeling `enum` as a class hierarchy:** The existing `Goal` enum (with associated values) cannot become `@Model` directly. The solution is to refactor it into a `Goal` base class and `TripGoal` / `ActivityGoal` subclasses — SwiftData supports class inheritance. `TripCollection` (a simple `RawRepresentable` enum without associated values) adds `Codable` conformance and is stored as a property.

**`@Attribute(.externalStorage)`:** Used for `thumbnailData: Data?` on `TripImage` to store large binary blobs outside the main SQLite store.

### Step 3: Define Model Relationships
Use `@Relationship(deleteRule:inverse:)` to declare to-many and to-one relationships. A `.cascade` delete rule on `Trip.activities` ensures `Activity` rows are deleted when their parent `Trip` is deleted. The `inverse:` parameter points back to the owning model's property (e.g., `\Activity.trip`). Once relationships are in place, the `DataSource` helper and `TripEditModel` are deleted entirely — SwiftData manages all persistence automatically.

### Setting Up the Container
Create `ModelContainer(for: Trip.self, Activity.self, TripImage.self, Goal.self, TripGoal.self, ActivityGoal.self)` and attach it to the app's `WindowGroup` with `.modelContainer(container)`. Seeding sample data runs against `container.mainContext` at startup if no data is already present. Changes are saved automatically (autosave is on by default).

### Updating the View Layer
Replace properties backed by `DataSource` with `@Query` property wrappers. Each view specifies its own targeted query:
- Simple sort: `@Query(sort: \Trip.name)`
- Filtered + sorted: `@Query(filter: #Predicate<Goal> { $0.isAchieved }, sort: \Goal.dateAchieved, order: .reverse)`
- `FetchDescriptor` with limit: `@Query(FetchDescriptor<Trip>(sortBy: [...], fetchLimit: 5))`
- Dynamic query constructed in `init`: assign `_trips = Query(filter: #Predicate<Trip> { ... })` inside the view's initializer

**Search:** A `SearchResultsListView` that takes `searchText` as a parameter reconstructs its `@Query` bindings in `init` based on whether the text is empty, switching between a recent-trips fetch and a `localizedStandardContains`-based predicate.

**Error handling:** `ModelContext` operations that throw (e.g., updating goal achievements) are caught and reported to the user using the `.alert(error:)` SwiftUI view modifier with a bound optional error.

**Re-enabling property observers:** After `@Model` adoption, `willSet`/`didSet` no longer fire reliably. Restore side effects by calling `withContinuousObservation(options: .didSet)` in the view's `init`, observing specific key paths with `event.matches(\Activity.name)` to conditionally update `dateEdited` and propagate computed state (e.g., `trip.isComplete`).

## APIs & Frameworks

**SwiftData — model definition**
- `@Model` macro — converts a class to a persistent model; generates `Observable` conformance
- `@Attribute(.codable)` — persist `Codable`-conforming types SwiftData cannot natively store
- `@Attribute(.externalStorage)` — store large `Data` blobs outside the main store
- `@Relationship(deleteRule:inverse:)` — declare relationships with cascade/nullify/deny rules
- `DeleteRule.cascade` — delete related objects when parent is deleted

**SwiftData — container & context**
- `ModelContainer(for:)` — creates the persistent store for a set of model types
- `ModelContainer.mainContext` — the main-thread `ModelContext`
- `ModelContext` — reads, writes, and deletes model instances; autosaves by default

**SwiftData — querying**
- `@Query(sort:)` / `@Query(filter:sort:order:)` — SwiftUI property wrapper for fetching
- `@Query(FetchDescriptor<T>(...))` — full `FetchDescriptor`-based query
- `FetchDescriptor<T>(sortBy:fetchLimit:)` — targeted fetch with limit
- `#Predicate<T> { }` macro — type-safe query predicate
- `SortDescriptor(\T.property, order:)` — sort descriptor
- Dynamic `@Query` in `init`: `_queryVar = Query(filter: #Predicate<T> { ... })`

**SwiftData — scene modifier**
- `.modelContainer(_:)` scene modifier — injects container into SwiftUI environment

**Swift Observation (new usage)**
- `withContinuousObservation(options: .didSet) { event in ... }` — replace `@Model`-incompatible property observers
- `ObservationTracking.Event.matches(\T.property)` — check which key path triggered the event
- `ObservationTracking.Token` — retain to keep observation active

**SwiftUI (error handling)**
- `.alert(error:) { }` view modifier — surface `Error`-conforming values as alerts

**MapKit (integration)**
- `MKMapItem.Identifier` — stored via `@Attribute(.codable)`

## Code Highlights

**`@Model` with `Codable` property and external attribute:**
```swift
@Model class Trip {
    var collection: TripCollection  // RawRepresentable + Codable enum — stored natively
    @Attribute(.codable) var mapItemIdentifier: MKMapItem.Identifier?
    @Relationship(deleteRule: .cascade, inverse: \Activity.trip)
    var activities: [Activity] = []
}
```

**App entry point with `ModelContainer`:**
```swift
@main struct WishlistApp: App {
    let container: ModelContainer = {
        let c = try! ModelContainer(for: Trip.self, Activity.self, TripImage.self, Goal.self, TripGoal.self, ActivityGoal.self)
        try? SampleData.seedIfNeeded(in: c.mainContext)
        return c
    }()
    var body: some Scene {
        WindowGroup { ContentView() }.modelContainer(container)
    }
}
```

**Dynamic query in view `init`:**
```swift
init(tripCollection: TripCollection, ...) {
    _trips = Query(filter: #Predicate<Trip> { $0.collection == tripCollection }, sort: \Trip.name)
}
```

**Re-enabling property observer side effects:**
```swift
activity.token = withContinuousObservation(options: .didSet) { event in
    if event.matches(\Activity.isComplete) {
        activity.dateEdited = .now
        activity.trip?.isComplete = activity.trip?.activities.allSatisfy { $0.isComplete } == true
    }
}
```

## Takeaways
- Follow the three-step adoption pattern — identify state, define schemas, define relationships — to make any SwiftData migration predictable and incremental.
- When an `enum` with associated values needs to become a model, refactor it into a class hierarchy (`@Model` base class + subclasses); SwiftData supports inheritance.
- Replace all property observers that have side effects with `withContinuousObservation(options: .didSet)` + `event.matches(\T.property)` after adopting `@Model`.
- Keep queries targeted: use `fetchLimit`, `#Predicate`, and per-view `@Query` declarations to minimize memory pressure and improve performance as the data set grows.

---
_Source: WWDC26 Session 275 page (abstract, chapter summaries, code samples, and resource links)._
