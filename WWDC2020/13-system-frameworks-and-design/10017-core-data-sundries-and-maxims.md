# Core Data: Sundries and Maxims
**WWDC20 · Session 10017** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10017/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14, watchOS 7

## Overview
This session covers three areas of Core Data optimization using the Earthquakes sample app (a Swift app ingesting USGS earthquake JSON feeds). The three pillars are: batch operations for fast data ingestion and mutation, tailored fetch requests for lower memory and higher performance, and notification APIs for efficiently reacting to persistent store changes from other processes or contexts.

The most significant new APIs are two additions to `NSBatchInsertRequest` — a `dictionaryHandler` block initializer and a `managedObjectHandler` block initializer — that reduce peak memory during large imports by processing data element by element instead of building a full array of dictionaries upfront. Combined with UPSERT behavior via `NSMergeByPropertyObjectTrumpMergePolicy`, these enable efficient idempotent data sync pipelines.

The notification modernization in iOS 14 adds two new ObjectID-based notifications (`didSaveObjectIDsNotification`, `didMergeChangesObjectIDsNotification`) and a Swift-friendly `NSManagedObjectContext.NotificationKey` enum for safe userInfo access, replacing stringly-typed keys. Remote change notifications allow apps to react to store changes from other processes (extensions, widgets, related apps) without polling.

## Key Topics

**NSBatchInsertRequest — Block Initializers (New)**
Two new initializers allow processing one element at a time. The block receives a mutable dictionary or managed object to fill in, returns `true` to stop (and trigger save), or returns `false` to continue with the next element. This eliminates the need to build a large array of dictionaries before calling the request, dramatically reducing peak memory for large ingestion operations.

Performance comparison for a large quake dataset:
- Regular `NSManagedObjectContext.save()`: ~60 seconds, 30 MB idle
- `NSBatchInsertRequest` with array of dictionaries: ~13 seconds, 25 MB idle
- `NSBatchInsertRequest` with block handler: ~11 seconds, lower peak memory

**UPSERT with Unique Constraints**
Setting a unique constraint on an entity attribute in the data model editor (e.g., `code` on `Quake`) plus setting `context.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy` before executing a batch insert produces SQL UPSERT behavior — conflicting rows are updated rather than duplicated or rejected.

**NSBatchUpdateRequest and NSBatchDeleteRequest**
`NSBatchUpdateRequest`: set `propertiesToUpdate` (dictionary of attribute names to new values) and `predicate` to conditionally update matching objects without fetching them. `NSBatchDeleteRequest`: always set a `fetchLimit` to prevent an unbounded write lock on the store when deleting large object graphs.

**Tailored Fetch Requests**
- `fetchBatchSize` on `NSFetchRequest`: only the first batch of objects is fully hydrated; subsequent batches load lazily as iterated, keeping memory low. The returned array is a "batched array" — unloaded objects appear as `NSManagedObjectID` until accessed.
- `propertiesToFetch`: limits which attributes are returned; reduces memory and parse time when only a subset of attributes is needed for display.
- `relationshipKeyPathsForPrefetching`: pre-fetch specified relationship key paths, avoiding per-object faults on traversal when relationships are known to be accessed.
- `NSFetchRequest.resultType`:
  - `.managedObjectResultType` — full managed objects (for `NSFetchedResultsController`)
  - `.managedObjectIDResultType` — lightweight `NSManagedObjectID` for cross-thread passing
  - `.dictionaryResultType` — read-only dictionaries; supports `NSExpressionDescription` for aggregates (`avg:`, `sum:`, `count:`, `min:`, `max:`) and `propertiesToGroupBy` for GROUP BY queries
  - `.countResultType` — returns a single count

**Modernized Notifications (New in iOS 14)**
Two new Swift-typed notification names on `NSManagedObjectContext`:
- `NSManagedObjectContext.didSaveObjectIDsNotification` — ObjectID-based counterpart to `didSaveObjectsNotification`
- `NSManagedObjectContext.didMergeChangesObjectIDsNotification` — ObjectID-based counterpart to `didChangeObjectsNotification`

New `NSManagedObjectContext.NotificationKey` enum replaces stringly-typed notification userInfo keys: `.insertedObjectIDs`, `.updatedObjectIDs`, `.deletedObjectIDs`, `.refreshedObjectIDs`, `.invalidatedObjectIDs`, and managed-object variants.

**Remote Change Notifications + Persistent History**
Enable both `NSPersistentStoreRemoteChangeNotificationPostOptionKey` and `NSPersistentHistoryTrackingKey` on the store description to receive `NSPersistentStoreRemoteChangeNotification` when any Core Data client (same app, extensions, widgets, related apps sharing the same container) modifies the persistent store. The notification's `userInfo` contains a `NSPersistentHistoryToken` for fetching the exact changes via `NSPersistentHistoryChangeRequest`.

**Tailored Persistent History Requests**
`NSPersistentHistoryChangeRequest.fetchHistory(after:)` combined with a custom `NSFetchRequest` on the `NSPersistentHistoryChange` entity allows filtering history to specific object IDs or date ranges, avoiding loading irrelevant history transactions.

## APIs & Frameworks

### Core Data — Batch Insert (Updated)
- `NSBatchInsertRequest(entity:dictionaryHandler:)` **[NEW in iOS 14]** — block processes one element at a time; return `true` to stop, `false` to continue
- `NSBatchInsertRequest(entity:managedObjectHandler:)` **[NEW in iOS 14]** — block populates an `NSManagedObject` instead of a dictionary
- `NSBatchInsertRequest.dictionaryHandler: ((inout [String: Any]) -> Bool)?` **[NEW]**
- `NSBatchInsertRequest.managedObjectHandler: ((inout NSManagedObject) -> Bool)?` **[NEW]**
- `NSBatchInsertRequest.resultType: NSBatchInsertRequestResultType` — `.statusOnly`, `.objectIDs`, `.count`
- `NSManagedObjectContext.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy` — enables UPSERT on unique-constrained attributes

### Core Data — Batch Update & Delete
- `NSBatchUpdateRequest(entityName:)` — bulk property update without fetching
  - `propertiesToUpdate: [AnyHashable: Any]` — dictionary of attribute name to new value
  - `predicate: NSPredicate?` — filter which objects are updated
- `NSBatchDeleteRequest(fetchRequest:)` — bulk delete without fetching
  - `fetchLimit` on the inner `NSFetchRequest` — always set to prevent unbounded write lock

### Core Data — Fetch Optimization
- `NSFetchRequest.fetchBatchSize: Int` — lazy hydration; unloaded objects are `NSManagedObjectID`
- `NSFetchRequest.propertiesToFetch: [Any]?` — limit returned attributes
- `NSFetchRequest.relationshipKeyPathsForPrefetching: [String]?` — prefetch specified relationship key paths
- `NSFetchRequest.resultType: NSFetchRequestResultType` — `.managedObjectResultType`, `.managedObjectIDResultType`, `.dictionaryResultType`, `.countResultType`
- `NSFetchRequest.propertiesToGroupBy: [Any]?` — GROUP BY for dictionary results
- `NSExpressionDescription` — computed column in dictionary results
- `NSExpression(forFunction:arguments:)` — `"avg:"`, `"sum:"`, `"count:"`, `"min:"`, `"max:"`

### Core Data — Notifications (New in iOS 14)
- `NSManagedObjectContext.didSaveObjectIDsNotification: Notification.Name` **[NEW]**
- `NSManagedObjectContext.didMergeChangesObjectIDsNotification: Notification.Name` **[NEW]**
- `NSManagedObjectContext.willSaveObjectsNotification: Notification.Name` **[NEW]**
- `NSManagedObjectContext.didSaveObjectsNotification: Notification.Name` **[NEW]**
- `NSManagedObjectContext.didChangeObjectsNotification: Notification.Name` **[NEW]**
- `NSManagedObjectContext.NotificationKey: String` **[NEW]** — enum cases: `.insertedObjectIDs`, `.updatedObjectIDs`, `.deletedObjectIDs`, `.refreshedObjectIDs`, `.invalidatedObjectIDs`, `.insertedObjects`, `.updatedObjects`, `.deletedObjects`, etc.

### Core Data — Remote Change Notifications + Persistent History
- `NSPersistentStoreRemoteChangeNotificationPostOptionKey` — store option to enable remote change notifications
- `NSPersistentHistoryTrackingKey` — store option to enable persistent history
- `NSPersistentStoreRemoteChangeNotification` — notification posted when any client modifies the store; userInfo contains `NSPersistentHistoryToken`
- `NSPersistentHistoryChangeRequest.fetchHistory(after:)` — fetch history after a token or date
- `NSPersistentHistoryChange.entityDescription(with:)` — entity description for filtering history changes
- `NSPersistentHistoryTransaction` — a recorded batch of changes
- `NSPersistentHistoryChange` — a single change within a transaction

## Code Highlights

Block-based batch insert (lower peak memory):
```swift
let batchInsert = NSBatchInsertRequest(
    entityName: "Quake",
    dictionaryHandler: { dictionary in
        guard blockCount < quakesBatch.count else { return true }  // true = stop
        dictionary = quakesBatch[blockCount] as! NSMutableDictionary
        blockCount += 1
        return false  // continue
    }
)
let result = try taskContext.execute(batchInsert) as! NSBatchInsertResult
```

UPSERT with unique constraint:
```swift
let moc = NSManagedObjectContext(concurrencyType: .privateQueueConcurrencyType)
moc.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
let result = try moc.execute(insertRequest) as! NSBatchInsertResult
```

Batch update with predicate:
```swift
let updateRequest = NSBatchUpdateRequest(entityName: "Quake")
updateRequest.propertiesToUpdate = ["validated": true]
updateRequest.predicate = NSPredicate(format: "%K > 2.5", "magnitude")
let result = try taskContext.execute(updateRequest) as! NSBatchUpdateResult
```

Batch delete with fetch limit:
```swift
let request = NSFetchRequest<Quake>(entityName: "Quake")
request.predicate = NSPredicate(format: "creationDate < %@", expirationDate)
let batchDelete = NSBatchDeleteRequest(fetchRequest: request)
request.fetchLimit = 1000  // prevent unbounded write lock
try moc.execute(batchDelete)
```

GROUP BY dictionary fetch (average magnitude per place):
```swift
let magnitudeExp = NSExpression(forKeyPath: "magnitude")
let avgExp = NSExpression(forFunction: "avg:", arguments: [magnitudeExp])
let avgDesc = NSExpressionDescription()
avgDesc.expression = avgExp
avgDesc.name = "average magnitude"
avgDesc.expressionResultType = .floatAttributeType

let fetch = NSFetchRequest<NSFetchRequestResult>(entityName: "Quake")
fetch.propertiesToFetch = [avgDesc, "place"]
fetch.propertiesToGroupBy = ["place"]
fetch.resultType = .dictionaryResultType
let results = try moc.fetch(fetch)
```

Enabling remote change notifications:
```swift
storeDesc.setOption(true as NSNumber, forKey: NSPersistentStoreRemoteChangeNotificationPostOptionKey)
storeDesc.setOption(true as NSNumber, forKey: NSPersistentHistoryTrackingKey)
```

Filtered persistent history fetch:
```swift
let changeDesc = NSPersistentHistoryChange.entityDescription(with: moc)
let request = NSFetchRequest<NSFetchRequestResult>()
request.entity = changeDesc
request.predicate = NSPredicate(format: "%K = %@",
    changeDesc?.attributesByName["changedObjectID"], targetObjectID)
let historyReq = NSPersistentHistoryChangeRequest.fetchHistory(after: Date.distantPast)
historyReq.fetchRequest = request
let results = try moc.execute(historyReq)
```

## Takeaways
- Use `NSBatchInsertRequest` with the new block initializers (`dictionaryHandler:` or `managedObjectHandler:`) for large data ingestion — they eliminate the need to build a full in-memory array and reduce peak memory significantly compared to both the array-of-dictionaries batch insert and traditional `save()`.
- Set `fetchBatchSize` on any `NSFetchRequest` driving a list UI to avoid hydrating thousands of managed objects upfront; combine with `propertiesToFetch` to load only the attributes actually displayed, reducing idle memory by 20-30%.
- Always set a `fetchLimit` on the `NSFetchRequest` inside an `NSBatchDeleteRequest` to prevent an unbounded write lock that blocks other readers and writers for the entire duration of a potentially large delete.
- Enable both `NSPersistentStoreRemoteChangeNotificationPostOptionKey` and `NSPersistentHistoryTrackingKey` together to eliminate polling for changes from extensions, widgets, and related apps — the remote change notification delivers a history token that pinpoints exactly what changed, and `NSPersistentHistoryChangeRequest` with a custom predicate can scope the history lookup to only the objects the current context cares about.

---
_Source: WWDC20 Session 10017 page (transcript, code samples, and resource links)._
