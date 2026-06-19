# Bring Core Data Concurrency to Swift and SwiftUI
**WWDC21 · Session 10017** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10017/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
This session covers two major sets of improvements to Core Data in 2021. The first is the adoption of Swift 5.5 structured concurrency: `NSManagedObjectContext`, `NSPersistentContainer`, and `NSPersistentStoreCoordinator` all gain new `perform(_:)` overloads decorated with `async`, enabling `try`/`await` usage instead of callbacks. The second set is SwiftUI enhancements: lazy entity resolution, dynamic configuration for fetch requests (predicates and sort descriptors changeable at runtime), and a new `SectionedFetchRequest` property wrapper that produces two-dimensional sectioned results.

The session uses an "Earthquakes" sample app (backed by USGS JSON feed data) as a running example, showing how each new API simplifies previously boilerplate-heavy patterns.

## Key Topics

### Core Data + Swift Concurrency
- New `async` `perform(_:)` overloads on `NSManagedObjectContext`, `NSPersistentContainer`, and `NSPersistentStoreCoordinator` **[NEW]**
- Closures can now `throw` and `return` values, eliminating hand-rolled result/error routing via captured optionals
- `ScheduledTaskType.immediate` (default): optimistically executes inline if already on the correct context; otherwise suspends until scheduled
- `ScheduledTaskType.enqueued`: always appends to the end of the context's work queue regardless of originating call site
- **Safety**: never return `NSManagedObject` instances from `perform`; use `NSManagedObjectID` or dictionary representations across contexts
- **Debugging tools**: Address Sanitizer, Thread Sanitizer (Xcode Diagnostics), Core Data concurrency debug flag (`-com.apple.CoreData.ConcurrencyDebug 1`)

### Swift API Ergonomics
- New short names for persistent store types **[NEW]**: `.sqlite`, `.xml`, `.binary`, `.inMemory` (old `NSSQLiteStoreType` etc. still work)
- New `NSAttributeDescription.AttributeType` enumeration **[NEW]** for all Core Data attribute types; enables type-safe model validation in tests

### SwiftUI: Lazy Entity Resolution
- `@FetchRequest` no longer requires the Core Data stack to be set up before views are initialized — entity lookup happens lazily at fetch time **[NEW]**
- Eliminates the need for "dummy" container properties on views just to guarantee early stack initialization

### SwiftUI: Dynamic Configuration for `FetchRequest`
- `FetchedResults.nsPredicate: NSPredicate?` **[NEW]** — change the fetch predicate at runtime
- `FetchedResults.sortDescriptors: [SortDescriptor<Element>]` **[NEW]** — change sort descriptors at runtime (also `nsSortDescriptors` for `NSSortDescriptor` compatibility)
- New `SortDescriptor` value type **[NEW]** using Swift key paths (`SortDescriptor(\Quake.time, order: .reverse)`)
- A `configuration` binding property exposes both `predicate` and `sortDescriptors` for use with external controls (e.g., toolbar menus)

### SwiftUI: `SectionedFetchRequest` and `SectionedFetchResults`
- New `@SectionedFetchRequest` property wrapper **[NEW]** — like `@FetchRequest` but groups results into sections
- Initialized with `sectionIdentifier:` key path — any `Hashable` property type (not limited to `String`)
- Returns `SectionedFetchResults<SectionIdentifier, Result>` **[NEW]**: a collection of `Section` objects, each itself a collection of `Result` objects
- Each section exposes `.id` (the section identifier value)
- Supports the same dynamic configuration properties as `FetchRequest`, plus `sectionIdentifier` key path change
- **Important**: always update `sectionIdentifier` and `sortDescriptors` together on a single local reference to `SectionedFetchResults` to avoid discontiguous sections

## APIs & Frameworks

### Core Data — Concurrency **[NEW]**
- `NSManagedObjectContext.perform<T>(schedule:_:) async throws -> T` **[NEW]**
  - `schedule: NSManagedObjectContext.ScheduledTaskType` — `.immediate` or `.enqueued`
- `NSPersistentContainer.performBackgroundTask<T>(_:) async throws -> T` **[NEW]**
- `NSPersistentStoreCoordinator.perform<T>(_:) async throws -> T` **[NEW]**

### Core Data — Enumeration API **[NEW]**
- `NSAttributeDescription.AttributeType` **[NEW]** — enumeration of attribute types (`.string`, `.integer16`, `.date`, `.uuid`, etc.)
- `NSPersistentStore.StoreType` short names **[NEW]**: `.sqlite`, `.xml`, `.binary`, `.inMemory`

### SwiftUI — FetchRequest **[NEW]**
- `FetchedResults<T>.nsPredicate: NSPredicate?` **[NEW]**
- `FetchedResults<T>.sortDescriptors: [SortDescriptor<T>]` **[NEW]**
- `FetchedResults<T>.nsSortDescriptors: [NSSortDescriptor]` **[NEW]**
- `SortDescriptor<Root>` **[NEW]** — value type using Swift key paths

### SwiftUI — SectionedFetchRequest **[NEW]**
- `@SectionedFetchRequest(sectionIdentifier:sortDescriptors:predicate:animation:)` **[NEW]**
- `SectionedFetchResults<SectionIdentifier, Result>` **[NEW]**
  - Subscript: collection of `SectionedFetchResults<SectionIdentifier, Result>.Section`
  - `.sectionIdentifier: KeyPath` — dynamic configuration
  - `.sortDescriptors`, `.nsSortDescriptors`, `.nsPredicate` — same dynamic config as `FetchedResults`

## Code Highlights

Async Core Data perform:
```swift
func importQuakes(from data: Data) async throws {
    let taskContext = newTaskContext()
    let quakePropertiesList = try await decode(data)
    try await taskContext.perform {
        let batchInsert = self.newBatchInsertRequest(with: quakePropertiesList)
        if let result = try? taskContext.execute(batchInsert) as? NSBatchInsertResult {
            // handle result
        }
    }
}
```

Dynamic sort descriptors on a FetchRequest:
```swift
quakes.sortDescriptors = [SortDescriptor(\Quake.magnitude, order: .reverse)]
```

Dynamic predicate on a FetchRequest:
```swift
quakes.nsPredicate = searchText.isEmpty
    ? nil
    : NSPredicate(format: "place CONTAINS %@", searchText)
```

SectionedFetchRequest:
```swift
@SectionedFetchRequest(
    sectionIdentifier: \.day,
    sortDescriptors: [SortDescriptor(\Quake.time, order: .reverse)])
private var quakes: SectionedFetchResults<String, Quake>

// In body:
ForEach(quakes) { section in
    Section(header: Text(section.id)) {
        ForEach(section) { quake in QuakeRow(quake: quake) }
    }
}
```

Safe atomic update of sectioned fetch (must use local reference):
```swift
let config = quakes
config.sectionIdentifier = sortBy.section
config.sortDescriptors = sortBy.descriptors
```

## Takeaways
- The new `async perform` APIs eliminate all manual error and result routing from Core Data concurrency code — use `try await taskContext.perform { ... }` and let Swift structured concurrency handle the rest.
- Never cross context boundaries with live `NSManagedObject` instances; use `NSManagedObjectID` or fetch-request dictionaries instead.
- `@SectionedFetchRequest` brings `NSFetchedResultsController`-style sectioning natively to SwiftUI with any `Hashable` section identifier type.
- Dynamic `sortDescriptors` and `nsPredicate` on `FetchedResults` / `SectionedFetchResults` let toolbar controls and search fields drive Core Data queries without restructuring view hierarchies.

---
_Source: WWDC21 Session 10017 page (abstract, transcript, and code samples)._
