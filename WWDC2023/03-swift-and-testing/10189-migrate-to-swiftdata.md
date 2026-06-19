# Migrate to SwiftData
**WWDC23 · Session 10189** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10189/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1

## Overview
This session is the practical migration guide for teams moving from Core Data to SwiftData. It covers three distinct paths: using Xcode's Managed Object Model Editor to generate SwiftData model classes from an existing `.xcdatamodeld` file, performing a complete replacement of the Core Data stack with a SwiftData stack, and running Core Data and SwiftData side-by-side in coexistence mode on the same persistent store for incremental adoption.

The session uses a "SampleTrips" trip-planning Core Data app as its running example throughout all three migration paths.

## Key Topics

### Generating SwiftData Model Classes from Core Data
- Xcode's Managed Object Model Editor provides an assistant: select the `.xcdatamodeld` file → menu bar → Editor → "Create SwiftData Code"
- Generates one Swift file per entity; each file contains a class conforming to `@Model` with properties matching the entity's attributes and relationships
- Generated files replace the NSManagedObject subclasses; the `.xcdatamodeld` can then be deleted for a complete migration
- Also works as a starting point reference even if code is further customized

### Complete SwiftData Adoption

**What to delete:**
- The Core Data `.xcdatamodeld` managed object model file
- The `PersistenceController` / `Persistence.swift` file that set up the `NSPersistentContainer`

**Setting up the SwiftData stack:**
- Add `.modelContainer(for: [ModelType.self, ...])` to the root `WindowGroup` in the `@main` App struct
- `modelContainer` creates the `ModelContainer` and injects a default `ModelContext` into the SwiftUI environment
- `@Environment(\.modelContext)` accesses the context from any view

**Object creation:**
- Core Data: `Trip(context: viewContext)` then set properties; requires passing the context
- SwiftData: `Trip(name:destination:startDate:endDate:)` — plain Swift initializer; then `modelContext.insert(trip)`

**Saving:**
- SwiftData features **implicit save**: automatically saves on UI lifecycle events and on a timer when the context has been modified
- Remove all explicit `context.save()` calls from Core Data code

**Fetching:**
- Replace `@FetchRequest` with `@Query(sort: \.startDate, order: .forward) var trips: [Trip]`
- `@Query` also supports predicates for filtered fetches

### Coexistence: Core Data + SwiftData on the Same Store
Coexistence runs two completely separate persistent stacks — one Core Data, one SwiftData — both pointed at the same on-disk store file.

**When to use coexistence:**
- App must maintain backward compatibility with iOS 16 / macOS Ventura (SwiftData requires iOS 17+)
- Resource constraints make a complete rewrite impractical
- New features can be built in SwiftData while legacy Core Data code continues to function
- UIKit/AppKit apps that need to adopt SwiftData incrementally

**Required setup for the Core Data stack:**
1. Set the persistent store URL explicitly to the shared store path before loading
2. Enable `NSPersistentHistoryTrackingKey = true` on the container description — SwiftData automatically enables persistent history tracking, but Core Data does not; mismatched tracking causes the store to open in read-only mode

**Class naming conflict:**
- Both stacks cannot have classes with the same name
- Rename the Core Data `NSManagedObject` subclass (e.g., `Trip` → `CDTrip`) while keeping the entity name unchanged in the model editor
- SwiftData class retains the original name (`@Model final class Trip`)

**Schema synchronization requirements:**
- Core Data and SwiftData schemas must stay in lock-step: every property and relationship added to SwiftData must also be added to the `NSManagedObjectModel`
- Entity version hashes must match; diverging hashes can trigger an unwanted migration that deletes data
- Track schema versions carefully; use versioned schemas in SwiftData (see "Model your schema with SwiftData")

**UIKit/AppKit without SwiftUI:**
- Option 1: Coexistence — UIKit binds to Core Data; new code uses SwiftData
- Option 2: Wrap SwiftData `@Model` classes as plain Swift classes and call them from UIKit code without SwiftUI property wrappers

## APIs & Frameworks

- **SwiftData** **[NEW]** – Swift-native persistence framework replacing Core Data boilerplate
- `@Model` macro **[NEW]** – designates a class as a SwiftData model type
- `ModelContainer` **[NEW]** – persistent backend; initialized with model types
- `.modelContainer(for:)` SwiftUI modifier **[NEW]** – sets up container and injects context
- `ModelContext` **[NEW]** – data context; accessed via `@Environment(\.modelContext)`
- `ModelContext.insert(_:)` **[NEW]** – inserts and begins tracking a new model object
- Implicit save **[NEW]** – SwiftData saves automatically on lifecycle events and context changes
- `@Query` property wrapper **[NEW]** – fetches and observes SwiftData objects; accepts `sort:` and predicate
- `SortDescriptor` – updated for native Swift keypaths; used in `@Query` and `FetchDescriptor`
- Managed Object Model Editor → "Create SwiftData Code" **[NEW Xcode feature]** – generates `@Model` classes from `.xcdatamodeld`
- **Core Data** – `NSPersistentContainer`, `NSManagedObjectContext`, `NSManagedObject`, `@FetchRequest` — the prior stack being replaced
- `NSPersistentHistoryTrackingKey` – must be set to `true` on Core Data stack when coexisting with SwiftData
- `persistentStoreDescriptions.first?.url` – set to shared store path for coexistence
- Versioned schemas – `VersionedSchema`, `SchemaMigrationPlan` (covered in "Model your schema with SwiftData", session 10195)

## Code Highlights

SwiftData app setup (replaces PersistenceController):
```swift
@main
struct TripsApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(
            for: [Trip.self, BucketListItem.self, LivingAccommodation.self]
        )
    }
}
```

Object creation — Core Data (before) vs SwiftData (after):
```swift
// Core Data
let newTrip = Trip(context: viewContext)
newTrip.name = name
newTrip.destination = destination

// SwiftData
let trip = Trip(name: name, destination: destination,
                startDate: startDate, endDate: endDate)
modelContext.insert(trip)
// No explicit save() needed — implicit save handles it
```

Fetching with @Query:
```swift
@Query(sort: \.startDate, order: .forward)
var trips: [Trip]
```

Core Data coexistence setup (persistent history + shared store URL):
```swift
let url = URL(fileURLWithPath: "/path/to/Trips.store")

if let description = container.persistentStoreDescriptions.first {
    description.url = url
    description.setOption(
        true as NSNumber,
        forKey: NSPersistentHistoryTrackingKey
    )
}
```

Avoiding class name collisions in coexistence:
```swift
// Core Data subclass renamed to avoid collision
class CDTrip: NSManagedObject { /* ... */ }

// SwiftData model keeps the user-facing name
@Model final class Trip { /* ... */ }
```

## Takeaways
- The Xcode "Create SwiftData Code" assistant accelerates migration by generating `@Model` classes directly from an existing `.xcdatamodeld`; this is the fastest starting point for any Core Data app
- Implicit save in SwiftData eliminates a common source of Core Data bugs (forgetting to call `save()`); remove all explicit `context.save()` calls when migrating
- Coexistence is a practical incremental strategy: Core Data handles existing features and old OS versions while new code is written in SwiftData on the same store; just enable persistent history tracking on the Core Data side and avoid class name collisions
- SwiftData requires iOS 17+; use coexistence to maintain iOS 16 support until your minimum deployment target can be raised

---
_Source: WWDC23 Session 10189 page (abstract, chapter summaries, code samples, and transcript)._
