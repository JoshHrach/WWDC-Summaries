# Using Core Data With CloudKit
**WWDC19 · Session 202** · [Watch](https://developer.apple.com/videos/play/wwdc2019/202/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
`NSPersistentCloudKitContainer` is introduced in iOS 13 as a new subclass of `NSPersistentContainer` that automatically syncs a Core Data store with a CloudKit private database. Adopting it can be as simple as changing one class name in an existing Core Data application. The container maintains a complete local replica of the CloudKit database, handles all scheduling and error recovery, and performs bi-directional serialization between `NSManagedObject` instances and `CKRecord` objects.

The session explains how to set up the container, how to configure multiple stores using `NSManagedObjectModel` configurations (local vs. cloud vs. shared), how Core Data maps its schema to CloudKit record types and field names, and how to use relationship-based data modeling to implement collaborative merge behavior without a custom conflict resolution strategy.

A new sample application (post/tag/attachment model) demonstrates end-to-end two-device sync including file attachments, and the session closes with a discussion of conflict-free replicated data types (CRDTs) and how Core Data relationships can approximate eventually-consistent, deterministic merge semantics.

## Key Topics

**NSPersistentCloudKitContainer**
Drop-in replacement for `NSPersistentContainer`. Provides: local replica of the private CloudKit database, automatic background scheduling of exports/imports, transparent `NSManagedObject` ↔ `CKRecord` serialization, and last-writer-wins conflict resolution by default. Configured via Xcode's "Use CloudKit" checkbox and requires iCloud + Background Modes (remote notifications) capabilities.

**Multiple Store Configurations**
`NSManagedObjectModel` configurations segment entities across separate SQLite files. A "Local" configuration holds high-frequency or non-synced data; a "Cloud" configuration holds entities backed by CloudKit; an optional "Shared" configuration points to a second iCloud container for cross-app data sharing.

**CloudKit Schema Mapping**
- Record types are prefixed with `CD_` to avoid collisions with developer-managed fields
- Each record includes `CD_entityName` to support entity inheritance via a single CloudKit record type
- Attributes appear as `CD_<attributeName>`; strings exceeding ~750 KB or records approaching the 1 MB limit are automatically externalized to `CD_<attributeName>_CKAsset`
- To-One relationships are stored as the UUID record name on the child record (`CD_post`)
- Many-To-Many relationships use a Core Data Mirrored Relationship (CDMR) join record containing entity names, record names, and relationship names for both sides

**Collaboration via Relationships**
Rather than writing to a flat string attribute (which produces last-writer-wins collisions), model collaborative content as a To-One relationship to a content entity. Multiple devices can independently insert child records without conflicting on the parent, achieving eventual consistency. Adding a timestamp or parent-contribution pointer enables causal ordering, approximating a CRDT.

**History Tracking Integration**
`NSPersistentHistoryTracking` lets the app inspect which objects changed in the background (due to CloudKit import) and decide whether to update the UI or notify the user.

## APIs & Frameworks

**Core Data**
- `NSPersistentCloudKitContainer` **[NEW]** — Core Data + CloudKit integration container
- `NSPersistentCloudKitContainerOptions` **[NEW]** — associates a store description with an iCloud container identifier
- `NSPersistentStoreDescription` — configures individual store files; `.configuration` property maps to model configurations
- `NSManagedObjectModel` configurations — used to partition entities across stores
- `NSFetchedResultsController` — scalable UI backed by local replica
- `NSQueryGenerationToken` / `managedObjectContext.queryGenerationToken` — stabilizes UI against background changes
- `NSPersistentHistoryTracking` (`NSPersistentStoreRemoteChange` notification) — detect CloudKit-imported changes
- `NSMergeByPropertyObjectTrumpMergePolicy` (last-writer-wins, automatic default)

**CloudKit** (managed internally by NSPersistentCloudKitContainer)
- `CKRecord` — serialization target for `NSManagedObject`
- `CKRecordZone` (custom zone per container) — holds all synced records
- `CKDatabase` (private) — default sync target
- `CKAsset` — storage for externalized large attribute values
- Push notifications (silent) — trigger background import

## Code Highlights

Minimum setup (replacing `NSPersistentContainer`):

```swift
// Before:
// let container = NSPersistentContainer(name: "Model")
// After:
let container = NSPersistentCloudKitContainer(name: "Model")
container.loadPersistentStores { _, error in
    if let error { fatalError(error.localizedDescription) }
}
container.viewContext.automaticallyMergesChangesFromParent = true
```

Multiple store configuration (local + cloud):

```swift
let localDesc = NSPersistentStoreDescription(url: localURL)
localDesc.configuration = "Local"   // no CloudKit options → stays on device

let cloudDesc = NSPersistentStoreDescription(url: cloudURL)
cloudDesc.configuration = "Cloud"
cloudDesc.cloudKitContainerOptions =
    NSPersistentCloudKitContainerOptions(containerIdentifier: "iCloud.com.example.app")

container.persistentStoreDescriptions = [localDesc, cloudDesc]
container.loadPersistentStores { _, error in ... }
```

Relationship-based collaborative content model (instead of flat string):

```swift
// NSManagedObject subclass
class PostContent: NSManagedObject {
    @NSManaged var text: String
    @NSManaged var createdAt: Date
    @NSManaged var post: Post           // To-One back to Post
}
// Post assembles content deterministically
var assembledContent: String {
    let sorted = contributions.sorted { $0.createdAt < $1.createdAt }
    return sorted.map(\.text).joined()
}
```

## Takeaways
- `NSPersistentCloudKitContainer` reduces a multi-thousand-line CloudKit sync implementation to a one-line class-name change, with automatic scheduling, error recovery, and serialization.
- Understanding the `CD_` prefixed schema and asset externalization is essential for consuming Core Data CloudKit records from non-Apple platforms or CloudKit Dashboard queries.
- Model collaborative content as relationships rather than flat attributes to avoid last-writer-wins data loss in multi-device editing scenarios.
- Combine with `NSPersistentHistoryTracking` and `NSFetchedResultsController` for a complete, reactive UI that responds correctly to background CloudKit imports.

---
_Source: WWDC19 Session 202 page (abstract, transcript, and resource links)._
