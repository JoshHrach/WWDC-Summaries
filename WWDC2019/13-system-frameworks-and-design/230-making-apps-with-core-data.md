# Making Apps with Core Data
**WWDC19 · Session 230** · [Watch](https://developer.apple.com/videos/play/wwdc2019/230/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
This session is a focused, best-practices tour of Core Data for iOS 13, covering the full stack from model design through advanced multi-coordinator synchronization and testing strategies. Three major new features headline the release: batch insertions, derived attributes, and remote change notifications with persistent history fetch requests.

The session uses a blogging sample app (posts, tags, media attachments) to demonstrate each concept concretely. It also covers integrating the new `NSFetchedResultsController` delegate methods that vend `NSDiffableDataSourceSnapshot` and `CollectionDifference` values directly, eliminating most of the historical boilerplate connecting Core Data to collection/table views.

Testing guidance emphasizes running integration tests with realistic data volumes, enabling Core Data concurrency debugging (`com.apple.CoreData.ConcurrencyDebug 1`), using SQLite in-memory stores for fast unit tests, and leveraging named in-memory stores to test remote change notification logic between multiple coordinators in-process.

## Key Topics
- **Core Data stack setup** — `NSPersistentContainer`, `NSManagedObjectModel`, `NSPersistentStoreCoordinator`, `NSManagedObjectContext`; query generations; `automaticallyMergesChangesFromParent`
- **Batch insertions** — `NSBatchInsertRequest` with array of dictionaries; respects unique constraints and default values; does not fire `NSManagedObjectContextDidSave` **[NEW]**
- **Derived attributes** — `NSDerivedAttributeDescription`; supports duplication, transformations (e.g. lowercase), aggregate functions (`@count`, `@sum`, etc.), and global functions (`now()`); defined in model editor or code **[NEW]**
- **NSFetchedResultsController + diffable data sources** — new delegate method vending `NSDiffableDataSourceSnapshot` for single-call collection view updates **[NEW]**
- **NSFetchedResultsController + CollectionDifference** — alternative delegate method for manual/sectioned use cases **[NEW]**
- **Persistent history fetch requests** — `NSPersistentHistoryTransaction` and `NSPersistentHistoryChange` now support `NSFetchRequest` predicates for filtered history queries **[NEW]**
- **Remote change notifications** — `NSPersistentStoreRemoteChangeNotificationPostOptionKey`; cross-coordinator save notifications delivered asynchronously; includes new history token **[NEW]**
- **Testing** — in-memory SQLite store (`/dev/null`), named in-memory store (`/dev/null/name`) for multi-coordinator tests, concurrency debug flag, address/thread/UBSan sanitizers

## APIs & Frameworks
- **Core Data**
  - `NSPersistentContainer` — stack encapsulation; `performBackgroundTask(_:)`
  - `NSManagedObjectContext`
    - `queryGenerationToken` / `setQueryGenerationFrom(.current)` **[query generations]**
    - `automaticallyMergesChangesFromParent: Bool`
    - `perform(_:)` / `performAndWait(_:)`
    - `mergeChanges(fromRemoteContextSave:into:)`
  - `NSFetchRequest` — `sortDescriptors`, `fetchBatchSize`, `predicate`
  - `NSBatchInsertRequest` **[NEW]**
    - `init(entity:objects:)` — dictionary-based bulk insert
    - `NSBatchInsertRequestResultType`
  - `NSDerivedAttributeDescription` **[NEW]** — set derivation expression (e.g. `"posts.@count"`)
  - `NSPersistentHistoryChangeRequest`
    - `init(fetchRequest:)` **[NEW]**
    - `fetchRequest: NSFetchRequest?` **[NEW]**
  - `NSPersistentHistoryTransaction`
    - `entityDescription(with:)` **[NEW]**
    - `fetchRequest()` **[NEW]**
  - `NSPersistentHistoryChange`
    - `entityDescription(with:)` **[NEW]**
    - `fetchRequest()` **[NEW]**
  - `NSPersistentStoreCoordinator.currentPersistentHistoryToken(fromStores:)` **[NEW]**
  - `NSPersistentStoreRemoteChangeNotificationPostOptionKey` **[NEW]** — option key for persistent store description
  - `NSPersistentStoreRemoteChangeNotification` **[NEW]** — notification name
  - `NSFetchedResultsController`
    - `controller(_:didChangeContentWith snapshot:)` — vends `NSDiffableDataSourceSnapshot` **[NEW]**
    - `controller(_:didChangeContentWith diff:)` — vends `CollectionDifference` **[NEW]**
  - Cascade deletion rules, unique constraints, default values
- **UIKit**
  - `NSDiffableDataSourceSnapshot` — snapshot type consumed by `UICollectionViewDiffableDataSource` / `UITableViewDiffableDataSource`
  - `UICollectionViewDiffableDataSource` **[NEW]**
  - `UITableViewDiffableDataSource` **[NEW]**
- **Foundation**
  - `CollectionDifference` — standard library diff type (SE-0240)
  - `NSPersistentHistoryToken`
- **Combine**
  - `NSManagedObject.publisher(for:)` — KVO publisher for reactive property binding in detail views
- **Testing infrastructure**
  - `com.apple.CoreData.ConcurrencyDebug 1` — launch argument for concurrency violation detection
  - SQLite in-memory store URL: `URL(fileURLWithPath: "/dev/null")`
  - Named in-memory store URL: `URL(fileURLWithPath: "/dev/null/SharedName")`

## Code Highlights

```swift
// Batch insert 1000 objects
let request = NSBatchInsertRequest(
    entity: Post.entity(),
    objects: payloadDictionaries  // [[String: Any]]
)
let result = try context.execute(request) as? NSBatchInsertResult
let success = result?.result as? Bool ?? false
```

```swift
// Derived attribute (postCount = posts.@count) in code
let derived = NSDerivedAttributeDescription()
derived.name = "postCount"
derived.attributeType = .integer64AttributeType
derived.derivationExpression = NSExpression(format: "posts.@count")
```

```swift
// FetchedResultsController → DiffableDataSource (single line)
func controller(_ controller: NSFetchedResultsController<NSFetchRequestResult>,
    didChangeContentWith snapshot: NSDiffableDataSourceSnapshot<String, NSManagedObjectID>) {
    dataSource.apply(snapshot)
}
```

```swift
// Remote change notifications
let description = container.persistentStoreDescriptions.first!
description.setOption(true as NSNumber,
    forKey: NSPersistentStoreRemoteChangeNotificationPostOptionKey)
NotificationCenter.default.addObserver(self,
    selector: #selector(storeRemoteChange(_:)),
    name: .NSPersistentStoreRemoteChange, object: container.persistentStoreCoordinator)
```

## Takeaways
- `NSBatchInsertRequest` replaces looping object creation for large imports — respects unique constraints and is dramatically faster.
- Derived attributes offload denormalization maintenance (counts, lowercase names, timestamps) into the model, eliminating error-prone manual bookkeeping.
- The new `NSFetchedResultsController` snapshot delegate method reduces collection view update glue to a single line.
- Remote change notifications + persistent history fetch requests provide a clean, near-real-time mechanism for keeping multiple coordinators synchronized without polling.

---
_Source: WWDC19 Session 230 page (abstract, chapter summaries, code samples, and resource links)._
