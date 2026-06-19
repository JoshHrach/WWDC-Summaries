# Meet SwiftData
**WWDC23 · Session 10187** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10187/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1

## Overview
SwiftData is a brand-new persistence framework built entirely for Swift, introduced at WWDC 2023. It uses Swift's macro system to generate schema definitions directly from Swift classes — no external model files, no .xcdatamodeld, no mapping model editors. Decorated with `@Model`, a plain Swift class becomes a fully persistent model type with change tracking, relationships, and CloudKit synchronization support.

SwiftData is built on Core Data under the hood and integrates seamlessly with SwiftUI. The `@Query` property wrapper fetches and observes data directly in views, and the new `ModelContainer` / `ModelContext` API replaces Core Data's `NSPersistentContainer` and `NSManagedObjectContext` with native Swift types. The `#Predicate` macro replaces `NSPredicate` with a fully type-checked, autocomplete-friendly predicate expression language.

## Key Topics

### @Model Macro
- Decorate any Swift class with `@Model` to declare it as a SwiftData model
- All stored properties become persisted attributes automatically: `String`, `Int`, `Float`, `Date`, structs, enums, `Codable` types, and collections
- Reference types between model classes become relationships
- No external schema files; the Swift code is the source of truth

### Schema Annotations
- `@Attribute(.unique)` – adds a uniqueness constraint on a property
- `@Attribute(.externalStorage)` – stores large binary data externally
- `@Relationship(.cascade)` – cascades deletes to related objects
- `@Relationship(inverse:)` – controls relationship inverse
- `@Transient` – excludes a property from persistence

### ModelContainer
- Persistent backend for model types; analogous to `NSPersistentContainer`
- Initialized with a list of model types or a `ModelConfiguration`
- `ModelConfiguration` accepts: custom URL, CloudKit container identifier, group container identifier, migration options
- SwiftUI integration: `.modelContainer(for:)` scene/view modifier propagates the container through the environment

### ModelContext
- Interface for fetch, insert, delete, save, and undo operations; analogous to `NSManagedObjectContext`
- Obtain from SwiftUI environment (`@Environment(\.modelContext)`), from `ModelContainer.mainContext`, or by instantiating directly
- `context.insert(_:)` – begins tracking a new model object
- `context.delete(_:)` – marks an object for deletion
- `context.save()` – commits pending changes to the container
- Autosave configurable via SwiftUI `.modelContainer(for:isAutosaveEnabled:)` modifier
- Undo/redo support via `.modelContainer(for:isUndoEnabled:)` modifier

### #Predicate Macro
- New in iOS 17; type-safe replacement for `NSPredicate`
- Uses Swift macros; Xcode provides autocomplete
- Expression operates on native Swift types and key paths
- Used with `FetchDescriptor` to filter fetched objects

### FetchDescriptor
- Replaces `NSFetchRequest` with a Swift-native generic type: `FetchDescriptor<T>`
- Parameters: `predicate`, `sortBy: [SortDescriptor]`, `fetchLimit`, `fetchOffset`, `includePendingChanges`, `relationshipKeyPathsForPrefetching`, `propertiesToFetch`
- Passed to `context.fetch(_:)` to retrieve model objects

### SortDescriptor Updates
- `SortDescriptor` now supports native Swift types and key paths (not just `NSObject` subclasses)
- Used directly in `FetchDescriptor` and `@Query`

### @Query Property Wrapper
- SwiftUI property wrapper that fetches and observes SwiftData objects directly in a view
- Automatically refreshes the view when observed model properties change (via the new Swift `Observable` framework)
- Accepts sort descriptor and predicate in its initializer
- Example: `@Query(sort: \.startDate, order: .reverse) var trips: [Trip]`

### Platform Integrations
- **CloudKit** – automatic iCloud sync via `ModelConfiguration` with CloudKit container ID
- **Widgets** – share SwiftData stores with WidgetKit extensions
- **Undo/redo** – enabled via SwiftUI container modifier

## APIs & Frameworks

- **SwiftData** **[NEW]** – Swift-native persistence framework
- `@Model` macro **[NEW]** – transforms a Swift class into a persistent model type
- `@Attribute` macro **[NEW]** – annotates properties with persistence metadata (`.unique`, `.externalStorage`, etc.)
- `@Relationship` macro **[NEW]** – controls relationship behavior (`.cascade`, `inverse:`)
- `@Transient` macro **[NEW]** – excludes a property from persistence
- `ModelContainer` **[NEW]** – persistent backend; replaces `NSPersistentContainer`
- `ModelConfiguration` **[NEW]** – customizes container (URL, CloudKit, group container, migrations)
- `ModelContext` **[NEW]** – data context for fetch/insert/delete/save; replaces `NSManagedObjectContext`
- `ModelContext.insert(_:)` **[NEW]**
- `ModelContext.delete(_:)` **[NEW]**
- `ModelContext.save()` **[NEW]**
- `#Predicate<T>` macro **[NEW]** – type-safe predicate; replaces `NSPredicate`
- `FetchDescriptor<T>` **[NEW]** – fetch request type; replaces `NSFetchRequest`
- `SortDescriptor` – updated to support native Swift keypaths **[UPDATED]**
- `@Query` property wrapper **[NEW]** – SwiftUI integration; fetches and observes SwiftData objects
- `.modelContainer(for:)` view/scene modifier **[NEW]** – SwiftUI environment setup
- `@Environment(\.modelContext)` **[NEW]** – access model context from SwiftUI view
- `ModelContainer.mainContext` **[NEW]** – main-actor-bound shared context

## Code Highlights

Model definition with annotations:
```swift
import SwiftData

@Model
class Trip {
    @Attribute(.unique) var name: String
    var destination: String
    var endDate: Date
    var startDate: Date

    @Relationship(.cascade) var bucketList: [BucketListItem]? = []
    var livingAccommodation: LivingAccommodation?
}
```

App setup with SwiftUI:
```swift
@main
struct TripsApp: App {
    var body: some Scene {
        WindowGroup { ContentView() }
            .modelContainer(for: [Trip.self, LivingAccommodation.self])
    }
}
```

Type-safe predicate and fetch:
```swift
let today = Date()
let tripPredicate = #Predicate<Trip> {
    $0.destination == "New York" &&
    $0.name.contains("birthday") &&
    $0.startDate > today
}
let descriptor = FetchDescriptor<Trip>(
    sortBy: SortDescriptor(\Trip.name),
    predicate: tripPredicate
)
let trips = try context.fetch(descriptor)
```

SwiftUI view with @Query:
```swift
struct ContentView: View {
    @Query(sort: \.startDate, order: .reverse) var trips: [Trip]
    @Environment(\.modelContext) var modelContext

    var body: some View {
        List(trips) { trip in /* ... */ }
    }
}
```

## Takeaways
- SwiftData replaces Core Data's entire authoring workflow: no `.xcdatamodeld`, no `NSManagedObject` subclasses, no fetch request boilerplate — just annotated Swift classes and macros.
- `#Predicate` is fully type-checked by the compiler with Xcode autocomplete; it catches errors at compile time that `NSPredicate` strings reveal only at runtime.
- `@Query` + `@Model` observes data changes automatically via Swift's new `Observable` framework, eliminating manual `NSFetchedResultsController` setup.
- SwiftData's `ModelConfiguration` supports CloudKit, group containers, and migration options, enabling drop-in iCloud sync and widget data sharing.

---
_Source: WWDC23 Session 10187 page (abstract, chapter summaries, code samples, and resource links)._
