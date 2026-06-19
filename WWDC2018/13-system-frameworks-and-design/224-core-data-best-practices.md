# Core Data Best Practices
**WWDC18 · Session 224** · [Watch](https://developer.apple.com/videos/play/wwdc2018/224/)

_Platforms:_ iOS 12, macOS Mojave 10.14, watchOS 5, tvOS 12

## Overview
This session covers modern Core Data architecture patterns and performance techniques for apps that have grown beyond their initial simplicity. The first half focuses on project structure: subclassing `NSPersistentContainer` to keep model bundles and store locations organized, designing view controllers with clean fetch-request and managed-object boundaries, and using `NSFetchedResultsController` and aggregate fetch requests for scalable data display.

The second half, presented by a second Core Data engineer, addresses concurrency correctness via query generations and persistent history tracking, batch operations for high-scale mutations, secure value transformers (`NSSecureUnarchiveFromDataTransformer`), SQL debugging flags, fetch indexes (including R-tree indexes for geo queries), and a testing strategy using in-memory stores and performance test scaffolding.

## Key Topics

### NSPersistentContainer Subclassing
- Subclassing `NSPersistentContainer` causes it to search the subclass's bundle for the `.xcdatamodeld` file — essential when the model lives in a framework target rather than the app bundle.
- Override `defaultDirectoryURL()` (class method) to redirect the store to a framework-specific directory, keeping framework files separate from the app's Documents folder.
- Use the subclass everywhere stack setup is needed; no other changes required.

### View Controller Design with Core Data
- List view controllers should receive an `NSFetchRequest` + `NSManagedObjectContext` (view context or main-queue context).
- Detail view controllers should receive an `NSManagedObject`.
- Background utility types should receive URLs or serialized data and use a background context.
- Pass parameters via `prepare(for:sender:)` (segues), property injection (nibs/storyboards), or explicit initializers.
- Always set `fetchBatchSize` or `fetchLimit` on fetch requests that may return an unbounded number of results.

### NSFetchedResultsController
- Available on all Apple platforms since macOS Sierra.
- Requires a fetch request and a managed object context; optional `sectionNameKeyPath` for section grouping.
- Drives list views with incremental updates; write a thin delegate adaptor between `NSFetchedResultsControllerDelegate` and the view.

### Aggregate Fetch Requests and Denormalization
- Use `NSFetchRequest` with `resultType = .dictionaryResultType`, `propertiesToGroupBy`, and `NSExpressionDescription` objects to run GROUP BY queries entirely in SQLite.
- Supported aggregate functions: count, average, sum, min, max, and scalar math/date functions via `NSExpression`.
- For very large datasets (50k+ rows), pre-aggregate into a separate entity (denormalization) and maintain counts in the `NSManagedObjectContextWillSave` notification — all updates occur in a single transaction.

### Query Generations
- Isolate a managed object context from concurrent writes: call `try context.setQueryGenerationFrom(.current)` once.
- Merge incoming changes selectively using `NSManagedObjectContextDidSave` notifications only when ready to manifest them in the UI.
- Requires WAL journal mode; SQLite stores only.

### Persistent History Tracking
- Introduced iOS 11 / macOS 10.13; opt in via `NSPersistentStoreDescription` option.
- `NSPersistentHistoryTransaction` — represents a committed batch of changes; provides `objectIDNotification()` to generate a save-like notification.
- `NSPersistentHistoryChange` — `changedObjectID`, `updatedProperties` — filter by entity and property to apply only UI-relevant changes, avoiding spurious scroll stutter.
- Batch delete/update operations bypass in-memory contexts and don't generate `NSManagedObjectContextDidSave`; use history tracking to observe and merge batch changes incrementally.

### Batch Operations
- `NSBatchDeleteRequest` — deletes objects matching a predicate directly in SQLite without faulting them into memory; O(1) memory regardless of result count (vs. O(n) for `NSManagedObject.delete()`).
- `NSBatchUpdateRequest` — updates attribute values for a predicate-matching set without loading objects.
- At 10 million rows, batch delete uses ~7% of the memory of a traditional delete.
- Combine with persistent history to propagate batch changes back to in-memory contexts.

### Secure Value Transformers
- `NSKeyedUnarchiveFromDataTransformer` (old default) is being replaced by `NSSecureUnarchiveFromDataTransformer` as the platform adopts secure coding.
- Set the transformer name on transformable attributes in the model editor or via `NSAttributeDescription.valueTransformerName`.
- Custom class types stored in transformable attributes must adopt `NSSecureCoding`.
- Xcode will warn in a future release when the legacy default is used; adopt now.

### SQL Debugging and Fetch Indexes
- `com.apple.CoreData.ConcurrencyDebug 1` — process argument; catches queue violations (cross-context object access) at runtime.
- `com.apple.CoreData.SQLDebug 1–4` — levels 1–3 log SQL statements and timing; level 4 adds query plan (`EXPLAIN`).
- SQLite `SQLITE_ENABLE_THREAD_ASSERTIONS` and `SQLITE_ENABLE_FILEASSERT` environment variables for additional correctness checking.
- Fetch indexes: add `NSFetchIndexDescription` to an entity in the model editor to create a covering B-tree index; eliminates in-memory sorts for `ORDER BY` on indexed columns.
- R-tree indexes: configure an `NSFetchIndexDescription` with type `.rTree` on latitude/longitude properties; use `NSExpression` predicates with bounding-box functions to hit the R-tree. Showed ~25% improvement on 100k row datasets.
- Use `sqlite3` CLI with `.expert` command to analyze a slow query and get index suggestions.

### Testing with Core Data
- Base test class using `/dev/null` as the store URL causes SQLite to create an in-memory store — very fast for small object graphs.
- Include at least one test that materializes the store on disk to verify the store can actually be opened.
- Build factory/scaffold methods to insert large amounts of test data in few lines of code.
- Wrap performance assertions in `measure {}` blocks; isolate setup/teardown from the measured code.
- Attach a minimal sample project or test suite to bug reports to communicate product requirements clearly.

## APIs & Frameworks

**Core Data**
- `NSPersistentContainer` — `defaultDirectoryURL()` class method override; `name` initializer; `viewContext`, `newBackgroundContext()`, `performBackgroundTask(_:)`
- `NSPersistentStoreDescription` — `url`, `type`, `shouldMigrateStoreAutomatically`, `shouldInferMappingModelAutomatically`; history tracking option key: `NSPersistentHistoryTrackingKey`
- `NSManagedObjectModel` — `mergedModel(from:)` for explicit bundle loading
- `NSManagedObjectContext` — `setQueryGenerationFrom(_:)`, `mergeChanges(fromContextDidSave:)`, `mergeChanges(fromRemoteContextSave:into:)`, `NSManagedObjectContextDidSave` notification, `NSManagedObjectContextWillSave` notification
- `NSFetchRequest` — `resultType` (`.managedObjectResultType`, `.dictionaryResultType`), `fetchBatchSize`, `fetchLimit`, `propertiesToGroupBy`, `havingPredicate`, `propertiesToFetch`
- `NSExpressionDescription` — aggregate expressions (count, sum, average, min, max)
- `NSExpression` — `init(forFunction:arguments:)`, scalar math and date functions
- `NSFetchedResultsController` — `sectionNameKeyPath`, `NSFetchedResultsControllerDelegate`
- `NSBatchDeleteRequest` — `init(fetchRequest:)`, `resultType` (`.resultTypeObjectIDs`)
- `NSBatchUpdateRequest` — `init(entityName:)`, `propertiesToUpdate`, `resultType`
- `NSPersistentHistoryTransaction` — `changes`, `objectIDNotification()`
- `NSPersistentHistoryChange` — `changedObjectID`, `updatedProperties`
- `NSPersistentHistoryChangeRequest` — `fetchHistory(after:)`, `deleteHistory(before:)`
- `NSFetchIndexDescription` — `name`, `elements`; type `.rTree` for geo queries **[NEW context for R-tree]**
- `NSFetchIndexElementDescription` — `property`, `collationType`
- `NSAttributeDescription.valueTransformerName` — set to `"NSSecureUnarchiveFromDataTransformer"` for transformable attributes
- `NSSecureUnarchiveFromDataTransformer` — new default transformer adopting `NSSecureCoding`

**Process Launch Arguments (Debugging)**
- `-com.apple.CoreData.ConcurrencyDebug 1`
- `-com.apple.CoreData.SQLDebug 1` (through `4`)
- `SQLITE_ENABLE_THREAD_ASSERTIONS=1` (environment variable)
- `SQLITE_ENABLE_FILEASSERT=1` (environment variable)

## Code Highlights

Subclassing `NSPersistentContainer` for framework bundle and custom store location:
```swift
class PhotosContainer: NSPersistentContainer {
    override class func defaultDirectoryURL() -> URL {
        return super.defaultDirectoryURL().appendingPathComponent("Photos")
    }
}
// PhotosContainer automatically searches its own bundle for the model.
```

Registering for context will-save to maintain denormalized counts:
```swift
NotificationCenter.default.addObserver(
    forName: .NSManagedObjectContextWillSave,
    object: viewContext, queue: nil) { notification in
    guard let context = notification.object as? NSManagedObjectContext else { return }
    for post in context.insertedObjects.compactMap({ $0 as? Post }) {
        post.day?.postCount += 1
    }
    for post in context.deletedObjects.compactMap({ $0 as? Post }) {
        post.day?.postCount -= 1
    }
}
```

Enabling query generation in one line:
```swift
try viewContext.setQueryGenerationFrom(.current)
```

Filtering persistent history changes to only UI-relevant entity/property updates:
```swift
func relevantChanges(in transaction: NSPersistentHistoryTransaction) -> Bool {
    guard let changes = transaction.changes else { return false }
    return changes.contains { change in
        change.changedObjectID.entity.name == "Post" &&
        change.updatedProperties?.contains(where: { $0.name == "imageData" || $0.name == "title" }) == true
    }
}
```

Batch delete with history-based merge:
```swift
let request = NSBatchDeleteRequest(fetchRequest: Post.fetchRequest())
request.resultType = .resultTypeObjectIDs
let result = try context.execute(request) as? NSBatchDeleteResult
let ids = result?.result as? [NSManagedObjectID] ?? []
NSManagedObjectContext.mergeChanges(fromRemoteContextSave: [NSDeletedObjectsKey: ids],
                                    into: [viewContext])
```

In-memory store for unit tests:
```swift
class CoreDataTestCase: XCTestCase {
    var container: NSPersistentContainer!
    override func setUp() {
        let description = NSPersistentStoreDescription()
        description.url = URL(fileURLWithPath: "/dev/null")
        container = PhotosContainer(name: "Photos")
        container.persistentStoreDescriptions = [description]
        container.loadPersistentStores { _, error in
            XCTAssertNil(error)
        }
    }
}
```

## Takeaways
- Subclass `NSPersistentContainer` and override `defaultDirectoryURL()` to keep model/store locations predictable as projects grow into multi-framework architectures.
- Always set `fetchBatchSize` or `fetchLimit`; use `NSFetchedResultsController` for live list views and aggregate fetch requests (GROUP BY) for summary/chart data before reaching for denormalization.
- Query generations and persistent history tracking together give fine-grained, opt-in UI update control — essential when background sync should not interrupt the user's current view.
- Batch operations scale with O(1) memory; combine with persistent history to propagate changes back without save notifications.

---
_Source: WWDC18 Session 224 page (abstract, full transcript, and resource links)._
