# Build Apps That Share Data Through CloudKit and Core Data
**WWDC21 · Session 10015** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10015/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15, watchOS 8

## Overview
This session introduces multi-user data sharing via `NSPersistentCloudKitContainer` in iOS 15. It covers three new capabilities: mirroring the CloudKit `.shared` database into a second persistent store, creating and accepting CloudKit shares using new container methods that pair with `UICloudSharingController`, and querying share metadata to conditionally enable/disable editing UI. A fourth topic covers encrypted Core Data attributes using a new model checkbox.

The session also demonstrates a recommended testing architecture: a `SharingProvider` protocol that abstracts all CloudKit sharing queries, with a `BlockBasedShareProvider` implementation that enables fast unit tests by injecting custom sharing logic without any CloudKit server calls.

## Key Topics

### CloudKit `.shared` Database Support
- `NSPersistentCloudKitContainer` previously only mirrored the `.private` database
- New: add a second `NSPersistentStoreDescription` with `cloudKitContainerOptions.databaseScope = .shared` **[NEW]**
- Both stores are accessible from a single `NSManagedObjectContext`; objects in the shared store came from other users sharing with you

### Record Zone Sharing vs. Hierarchical Sharing
- `NSPersistentCloudKitContainer` uses **Record Zone Sharing** (new CloudKit feature): all shared records live in a dedicated `CKRecordZone` identified by a single `CKShare` record at the zone level
- No root record required; the zone itself is the unit of sharing
- Owners can create and modify records in their shared zones; participants can read (or write, per permissions) in zones shared with them
- Zone assignment: pass a non-nil `CKShare` to `share(_:to:completion:)` to place objects in a specific existing share/zone

### Creating Shares
- `NSPersistentCloudKitContainer.share(_:to:completion:)` **[NEW]** — identifies all objects needing sharing and creates a `CKShare` if needed
- Designed to pair with `UICloudSharingController`'s create-share callback
- Pass the resulting `CKShare` and `CKContainer` to the sharing controller's completion block
- Set share permissions (read/write vs. read-only) via `UICloudSharingController`'s options before presenting

### Accepting Shares
- `NSPersistentCloudKitContainer.acceptShareInvitations(from:into:completion:)` **[NEW]** — call from `UIApplicationDelegate.application(_:userDidAcceptCloudKitShareWith:)`
- Pass incoming `CKShareMetadata` and the shared persistent store
- Container syncs all shared objects into the local store automatically after acceptance

### Share Metadata API for UI Customization
- `NSPersistentCloudKitContainer.fetchShares(matching:)` **[NEW]** — returns `[NSManagedObjectID: CKShare]` for a set of object IDs
- Existing methods (introduced with `.public` database support, WWDC20): `canUpdateRecord(forManagedObjectWith:)`, `canDeleteRecord(forManagedObjectWith:)`, `isRecord(for:shareable:)`
- Typical UI customizations needed: decorate shared-object cells, disable edit/delete buttons for read-only participants, display participant list with roles and acceptance status

### SharingProvider Protocol (Testing Architecture)
- Define a `SharingProvider` protocol with methods: `isShared(object:)`, `participants(for:)`, `shares(matching:)`, `canEdit(object:)`, `canDelete(object:)`
- `CoreDataStack` conforms to `SharingProvider` using the real CloudKit APIs
- `BlockBasedShareProvider` (test-only) conforms via injectable blocks — no network calls needed to test all sharing-dependent UI paths
- Inject via property on view controllers for testability

### Encrypted Core Data Attributes
- New `Allows Cloud Encryption` checkbox in Xcode data model editor **[NEW]** — stores attribute value in `CKRecord.encryptedValues` payload instead of plain fields
- Programmatic API: `NSAttributeDescription.allowsCloudEncryption: Bool` **[NEW]**
- Encryption is an at-schema-introduction-time decision — cannot encrypt an already-deployed field, or unencrypt an encrypted field, once pushed to production
- Use `NSPersistentCloudKitContainer.initializeSchema()` before production deployment to verify field types

## APIs & Frameworks

### Core Data / NSPersistentCloudKitContainer **[NEW]**
- `NSPersistentCloudKitContainerOptions.databaseScope: CKDatabase.Scope` **[NEW]** — `.private`, `.shared`, or `.public`
- `NSPersistentCloudKitContainer.share(_ managedObjects: [NSManagedObject], to share: CKShare?, completion:)` **[NEW]**
  - Completion: `(Set<NSManagedObjectID>?, CKShare?, CKContainer?, Error?) -> Void`
- `NSPersistentCloudKitContainer.acceptShareInvitations(from metadata: CKShareMetadata, into persistentStore: NSPersistentStore, completion:)` **[NEW]**
- `NSPersistentCloudKitContainer.fetchShares(matching objectIDs: [NSManagedObjectID]) throws -> [NSManagedObjectID: CKShare]` **[NEW]**
- `NSPersistentCloudKitContainer.canUpdateRecord(forManagedObjectWith:) -> Bool`
- `NSPersistentCloudKitContainer.canDeleteRecord(forManagedObjectWith:) -> Bool`
- `NSAttributeDescription.allowsCloudEncryption: Bool` **[NEW]**

### UIKit / CloudKit
- `UICloudSharingController` — system sharing sheet; create-share callback pairs with `container.share(_:to:completion:)`
- `UIApplicationDelegate.application(_:userDidAcceptCloudKitShareWith: CKShareMetadata)` — pass metadata to `acceptShareInvitations`
- `CKShare.Participant` — `.role: CKShare.ParticipantRole`, `.permission: CKShare.ParticipantPermission`, `.acceptanceStatus: CKShare.ParticipantAcceptanceStatus`

## Code Highlights

Add shared persistent store description:
```swift
let sharedStoreURL = storesURL.appendingPathComponent("shared.sqlite")
guard let sharedStoreDescription = privateStoreDescription.copy() as? NSPersistentStoreDescription else { fatalError() }
sharedStoreDescription.url = sharedStoreURL
let sharedOptions = NSPersistentCloudKitContainerOptions(containerIdentifier: containerIdentifier)
sharedOptions.databaseScope = .shared
sharedStoreDescription.cloudKitContainerOptions = sharedOptions
container.persistentStoreDescriptions.append(sharedStoreDescription)
```

Create share paired with UICloudSharingController:
```swift
let sharingController = UICloudSharingController { controller, completion in
    container.share([post], to: nil) { objectIDs, share, ckContainer, error in
        if let share { share[CKShare.SystemFieldKey.title] = post.title }
        completion(share, ckContainer, error)
    }
}
present(sharingController, animated: true)
```

Accept incoming share:
```swift
func application(_ app: UIApplication, userDidAcceptCloudKitShareWith metadata: CKShareMetadata) {
    let container = AppDelegate.sharedAppDelegate.coreDataStack.persistentContainer
    container.acceptShareInvitations(from: metadata, into: sharedStore) { _, error in
        if let error { print("Accept error: \(error)") }
    }
}
```

SharingProvider protocol for testable UI:
```swift
protocol SharingProvider {
    func isShared(object: NSManagedObject) -> Bool
    func participants(for object: NSManagedObject) -> [CKShare.Participant]
    func shares(matching objectIDs: [NSManagedObjectID]) throws -> [NSManagedObjectID: CKShare]
    func canEdit(object: NSManagedObject) -> Bool
    func canDelete(object: NSManagedObject) -> Bool
}
```

## Takeaways
- Enabling sharing requires adding just one second persistent store description (`.shared` scope) and calling two new methods: `share(_:to:completion:)` for creating shares and `acceptShareInvitations(from:into:completion:)` for accepting them.
- Record Zone Sharing (new in CloudKit) eliminates the need for a root record — the entire zone is shared as a unit, which simplifies NSPersistentCloudKitContainer's internal management.
- Building a `SharingProvider` protocol and injecting a `BlockBasedShareProvider` in tests enables full coverage of all sharing-dependent UI branches without any network calls — essential given the complexity sharing adds to every list and detail view.
- Encrypted attributes (`allowsCloudEncryption`) are a one-way, at-introduction-time decision — plan the schema carefully before promoting to production, and always call `initializeSchema()` first.

---
_Source: WWDC21 Session 10015 page (abstract, transcript, and code samples)._
