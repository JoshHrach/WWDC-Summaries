# Sync a Core Data Store with the CloudKit Public Database
**WWDC20 · Session 10650** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10650/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7, tvOS 14

## Overview
`NSPersistentCloudKitContainer` gains support for the CloudKit public database in iOS 14 and Xcode 12 with a single new property: `databaseScope`. Setting it to `.public` on `NSPersistentCloudKitContainerOptions` points a Core Data store at the public CloudKit database instead of the private one, giving all users of an app a shared, readable dataset (high scores, templates, shared catalogs) that can also be contributed to when an iCloud account is present.

Because the CloudKit public database does not support push notifications or `CKFetchRecordZoneChangesOperation`, `NSPersistentCloudKitContainer` falls back to polling via `CKQueryOperation`. Polls occur on app launch and roughly every 30 minutes thereafter. This approach fetches each entity type with a separate network request, so keeping the managed object model's public-database configuration lean reduces server load and latency.

Deletions behave differently in the public database: records are removed immediately without leaving a tombstone, so deletes do not propagate to other devices. The recommended pattern is to mark records as "trashed" via an update, filter them out of the UI, and purge them later, rather than hard-deleting them. New API methods—`canUpdateRecord(forManagedObjectWithID:)`, `canDeleteRecord(forManagedObjectWithID:)`, and `canModifyObjects(in:)`—provide efficient per-object and per-store permission checks that are safe to use in the UI layer.

## Key Topics
- **One-line adoption** — set `NSPersistentCloudKitContainerOptions.databaseScope = .public` on the store description
- **CloudKit Dashboard configuration** — add `recordName` and `modifiedAt` indexes to every record type before the public database can be queried
- **Schema initialization** — use `NSPersistentCloudKitContainer`'s schema initialization process to create a complete local mirror
- **Accounts and ownership** — reads are allowed signed-out; writes require sign-in; only the record creator can modify a record
- **Permission APIs** — `canUpdateRecord(forManagedObjectWithID:)`, `canDeleteRecord(forManagedObjectWithID:)`, `canModifyObjects(in:)` replace manual CloudKit account checks
- **Import via polling** — public database uses `CKQueryOperation` per entity type; polls on launch and every ~30 minutes
- **Delete propagation limitation** — public database has no tombstones; use soft-delete (trash flag + update) instead of hard delete
- **Managed object model configurations** — restrict public-database entities to a dedicated configuration to minimise polling requests
- **Mixing public and private stores** — private user data (saved game state) stays in the private database while shared data (scores) goes in the public database

## APIs & Frameworks

**Core Data / NSPersistentCloudKitContainer**
- `NSPersistentCloudKitContainer` — existing class; gains public database support **[NEW]**
- `NSPersistentCloudKitContainerOptions` — options object for configuring CloudKit integration
- `NSPersistentCloudKitContainerOptions.databaseScope` **[NEW]** — set to `.public` or `.private` (default) to choose the CloudKit database
- `NSPersistentCloudKitContainer.canUpdateRecord(forManagedObjectWithID:)` **[NEW]** — returns Bool; checks whether the current user can update the given object considering account state and store configuration
- `NSPersistentCloudKitContainer.canDeleteRecord(forManagedObjectWithID:)` **[NEW]** — returns Bool; returns `false` for records in the public database (no tombstone support), guiding soft-delete patterns
- `NSPersistentCloudKitContainer.canModifyObjects(in:)` **[NEW]** — returns Bool; checks whether any objects in the given persistent store are mutable (useful for table/collection view edit controls)
- `NSPersistentStoreDescription` — used to configure the store path, options, and history tracking
- `NSPersistentStoreDescription.setOption(_:forKey:)` — used to enable persistent history tracking and remote change notifications

**CloudKit**
- `CKDatabase` — represents private, public, and shared databases
- `CKDatabase.Scope` — `.public`, `.private`, `.shared` **[NEW scope used with Core Data]**
- `CKQueryOperation` — used by `NSPersistentCloudKitContainer` to poll the public database per entity type
- `CKFetchRecordZoneChangesOperation` — used for the private database (not available for public database)
- `CKRecordZone.Capabilities` — `fetchChanges` capability absent on public database zones
- `CKRecord` — the underlying CloudKit record type
- `CKRecordZone` — record container in CloudKit

**Foundation / Notifications**
- `NSPersistentStoreRemoteChangeNotification` — notification posted when remote changes are imported
- `NSPersistentHistoryTrackingKey` — option key to enable persistent history

## Code Highlights

Configuring `NSPersistentCloudKitContainer` for the public database:
```swift
let storeDescription = NSPersistentStoreDescription(url: storeURL)
storeDescription.setOption(true as NSNumber, forKey: NSPersistentHistoryTrackingKey)
storeDescription.setOption(true as NSNumber,
    forKey: NSPersistentStoreRemoteChangeNotificationPostOptionKey)

let options = NSPersistentCloudKitContainerOptions(containerIdentifier: "iCloud.com.example.MyApp")
options.databaseScope = .public   // NEW
storeDescription.cloudKitContainerOptions = options

container.persistentStoreDescriptions = [storeDescription]
container.loadPersistentStores { ... }
```

Checking mutability before showing edit controls:
```swift
let canEdit = container.canUpdateRecord(forManagedObjectWithID: objectID)
editButton.isEnabled = canEdit
```

Soft-delete pattern for public database records:
```swift
if container.canDeleteRecord(forManagedObjectWithID: objectID) {
    context.delete(object)
} else {
    object.isTrashed = true  // update in place; filter with a predicate
}
```

## Takeaways
- Adopting the public database requires only one new line (`options.databaseScope = .public`) plus adding `recordName` and `modifiedAt` indexes in the CloudKit Dashboard for every record type.
- Import from the public database uses polling with `CKQueryOperation`—once on launch and every ~30 minutes—so freshness expectations differ significantly from the private database.
- Hard deletes do not propagate in the public database; use a "trashed" flag + soft delete and filter it in fetch request predicates, then purge asynchronously.
- Use `canUpdateRecord(forManagedObjectWithID:)`, `canDeleteRecord(forManagedObjectWithID:)`, and `canModifyObjects(in:)` to drive UI state rather than writing manual CloudKit account-checking code.

---
_Source: WWDC20 Session 10650 page (abstract, chapter summaries, code samples, and resource links)._
